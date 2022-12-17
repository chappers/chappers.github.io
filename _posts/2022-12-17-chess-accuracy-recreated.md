---
layout: post
category:
tags:
tagline:
---

Lichess published their accuracy model: https://lichess.org/page/accuracy

Which interpolates a guess for winning probabilities and uses intepolation with harmonic mean to determine overall accuracy. 

Here is some code to reproduce:

```py
from stockfish import Stockfish
import numpy as np
import tqdm
from scipy.stats import hmean, gmean

class ChessFeatureEngine:
    def __init__(self, game):
        self.game = game  # the python-chess game object

    def generate_features(self):
        # features are a list of dicts
        white = []
        black = []
        summary = {}
        tau = 0.1

        board = self.game.board()
        white_move = True
        features = None
        for move in tqdm.tqdm(self.game.mainline_moves()):
            fen = board.fen()
            player_move = str(move)
            features = FeatureFromFEN.generate_features(fen)

            if white_move:
                prev_features = {} if len(white) == 0 else white[-1]
                features['accuracy'] = FeatureFromFEN.accuracy(features.get('win_percentage'), prev_features.get('win_percentage'))
                features['polyak_accuracy'] = FeatureFromFEN.polyak_accuracy(features['accuracy'], prev_features.get('polyak_accuracy'))
                white.append(features)
            else:
                prev_features = {} if len(black) == 0 else black[-1]
                features['accuracy'] = FeatureFromFEN.accuracy(features.get('win_percentage'), prev_features.get('win_percentage'))
                features['polyak_accuracy'] = FeatureFromFEN.polyak_accuracy(features['accuracy'], prev_features.get('polyak_accuracy'))
                black.append(features)
            board.push(move)
            white_move = not white_move
        
        white_accuracy = np.array([x['accuracy'] for x in white if x['accuracy'] is not None])
        black_accuracy = np.array([x['accuracy'] for x in black if x['accuracy'] is not None])

        rescale_accuracy = max(white_accuracy.max(), black_accuracy.max(), 100.0)/100.0

        summary['white_accuracy'] = hmean(white_accuracy/rescale_accuracy)
        summary['white_gmean_accuracy'] = gmean(white_accuracy/rescale_accuracy)
        summary['white_polyak_accuracy'] = white[-1]['polyak_accuracy']
        
        summary['black_accuracy'] = hmean(black_accuracy/rescale_accuracy)
        summary['black_gmean_accuracy'] = gmean(black_accuracy/rescale_accuracy)
        summary['black_polyak_accuracy'] = black[-1]['polyak_accuracy']

        return white, black, summary


class FeatureFromFEN:
    stockfish = Stockfish(path="C:/Users/SR55E/Documents/stockfish-11-win/Windows/stockfish_20011801_32bit.exe", depth=10)
    
    # generate features from FEN - this is to be averaged out later.
    # or set by weighted number, output is statistics + weights
    @classmethod
    def generate_features(cls, fen):
        # generates features for this particular FEN only, returns a dictionary
        features = {}
        cls.stockfish.set_fen_position(fen)
        is_white = cls.stockfish.get_fen_position().split(" ")[1] == "w"

        features['win_percentage'] = cls.win_percentage(cls.stockfish) if is_white else 100-cls.win_percentage(cls.stockfish)
        return features

    @classmethod
    def win_percentage(cls, stockfish):
        sf_evaluation = stockfish.get_evaluation()
        if sf_evaluation['type'] == 'cp':
            centipawns = sf_evaluation['value']
            win_proba = 50+50 * (2/(1+np.exp(-0.00368208 * centipawns)) - 1)
        else:
            win_proba = (int(sf_evaluation['mate'] > 0) - 0.5)*200.0
        return win_proba

    @classmethod
    def accuracy(cls, win_percentage_after, win_percentage_before=None):
        if win_percentage_before is None:
            return None
            
        accuracy_output = 103.1668 * np.exp(-0.04354 * (win_percentage_before - win_percentage_after))-3.1669
        accuracy_output += 1 # uncertainty bonus
        return np.clip(accuracy_output, 0, 100)

    @classmethod
    def polyak_accuracy(cls, accuracy, prev_polyak_accuracy=None):
        if prev_polyak_accuracy is None:
            return accuracy
        return hmean([accuracy, prev_polyak_accuracy])

if __name__ == "__main__":
    import chess.pgn
    pgn = open("{fname}.pgn")
    game = chess.pgn.read_game(pgn)

    chess_features = ChessFeatureEngine(game)
    white, black, summary = chess_features.generate_features()
```

Enjoy!
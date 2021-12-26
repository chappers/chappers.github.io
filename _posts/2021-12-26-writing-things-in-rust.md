---
layout: post
category : 
tags : 
tagline: 
---

In my journey to learning rust, I thought I'll pull together a simple program and demonstrate the parallels between Rust `struct` and Python `objects` as well as how they can be updated.

*  `structs` and `dataclass` objects operate in a similar way

```py
from dataclasses import dataclass
from enum import Enum, auto
from typing import Dict


class Color(Enum):
    RED = auto()
    BLUE = auto()
    GREEN = auto()


@dataclass
class Card(object):
    price: Dict[Color, int]
    color: Color


if __name__ == "__main__":
    print(Card(price = {Color.RED: 2}, color = Color.RED))
```


```rs
use std::collections::HashMap;

enum Color {
    Red,
    Blue,
    Green,
}

#[derive(Debug)]
struct Card {
    price: HashMap<Color, i32>,
    color: Color,
}

fn main() {
    let card = Card {
        price: HashMap::from([(Color::Red, 2)]),
        color: Color::Red,
    };
    println!("{:?}", card);
}
```

Already there are a lot of similarities in how the programs are structured and wirtten out. In both settings we've only used the standard libraries, and we can see that there is similar lines of code used in both examples. 

Let's suppose we have a wallet which holds cards, and we want to update the cards and coins which our wallet can hold. How would we add methods to enable this?


```py
from pydantic import BaseModel
from enum import Enum, auto
from typing import Dict, List


class Color(Enum):
    RED = auto()
    BLUE = auto()
    GREEN = auto()


class Card(BaseModel):
    price: Dict[Color, int]
    color: Color


class Wallet(BaseModel):
    cards: List[Card] = []
    coins: Dict[Color, int] = {}

    def add_card(self, card):
        self.cards = self.cards + [card]

    def add_coin(self, color, value):
        self.coins[color] = self.coins.get(color, 0) + value


if __name__ == "__main__":
    wallet = Wallet(cards=[Card(price={Color.RED: 1}, color=Color.RED)], coins = {Color.RED:3})
    print(wallet)
    wallet.add_coin(Color.BLUE, 10)
    print(wallet)

```

Rust variation

```rs
use std::collections::HashMap;

#[derive(Debug, PartialEq, Eq, Hash, Clone)]
enum Color {
    Red,
    Blue,
    Green,
}

#[derive(Debug)]
struct Card {
    price: HashMap<Color, i32>,
    color: Color,
}

#[derive(Debug)]
struct Wallet {
    cards: Vec<Card>,
    coins: HashMap<Color, i32>,
}

impl Wallet {
    fn add_card(&mut self, card: Card) {
        self.cards.push(card);
    }

    fn add_coin(&mut self, coin: &Color, value: i32) {
        // update coins
        if let Some(elements) = self.coins.get_mut(coin) {
            *elements = *elements + value;
        } else {
            self.coins.insert(coin.to_owned(), value);
        }
    }
}


fn main() {
    let card = Card {
        price: HashMap::from([(Color::Red, 2)]),
        color: Color::Red,
    };

    let mut wallet = Wallet {
        cards: vec![card],
        coins: HashMap::from([])
    };
    println!("{:?}", wallet);
    wallet.add_coin(&Color::Red, 1);
    println!("{:?}", wallet);
    wallet.add_coin(&Color::Red, 1);
    println!("{:?}", wallet);
}
```

There is some more additional boilerplate code, we need to take care of which is accomplished through adding traits via `derive`. In addition we use the dereference operator `*` to access `elements` to update the value. 
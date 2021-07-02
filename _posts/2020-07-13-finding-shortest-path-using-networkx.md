---
layout: post
category : 
tags : 
tagline: 
---

One of the things I've been experimenting with is imitation learning in RL. As such one aspect is how to imitate experts which take the optimal route. Since I don't really care about writing a BFS algorithm (or A* etc), what's the easiest out-of-the-box method to use? Surprisingly, there aren't many examples online, so in this blog post I'll attempt to demonstrate how we can do this using `networkx`

## Setting up our maze

We'll first setup our maze in a `gym`-like environment, just so that we have a consistent interface to everyone else. I actually did this in the `gym-minigrid` environment, but I'll leave that for a discussion on another day. 

```py
import networkx as nx
import numpy as np

# create a maze structure that is firstly open
def get_node_number(r, c, rows, cols):
    temp = np.zeros((rows, cols))
    temp[r, c] = 1
    return np.argwhere(temp.flatten() == 1)[0][0]

def get_node_number_rev(node, rows, cols):
    temp = np.zeros((rows, cols)).flatten()
    temp[node] = 1
    temp = temp.reshape(rows, cols)
    return list(np.argwhere(temp == 1).flatten())

def open_room(rows, cols):
    num_cells = rows*cols
    adj_matrix = np.zeros((num_cells, num_cells))
    for r in range(rows):
        for c in range(cols):
            for dx, dy in [(-1, 0), (1, 0), (0, 1), (0, -1)]:
                r0, c0 = r+dx, c+dy
                if r0 >= 0 and r0 < rows and c0 >= 0 and c0 < cols:
                    adj_matrix[get_node_number(r, c, rows, cols), get_node_number(r0, c0, rows, cols)] = 1
    return adj_matrix

def generate_maze(rows, cols):
    # print("Generating maze of size: ", rows * 2+1, "x", cols*2+1)
    # rows = int(np.ceil(shape[0]/2))
    # cols = int(np.ceil(shape[1]/2))

    adj_matrix = open_room(rows, cols)
    adj_matrix = adj_matrix  * np.abs(np.random.uniform(size=adj_matrix.shape))
    G = nx.from_numpy_matrix(adj_matrix)
    T = nx.minimum_spanning_tree(G)
    maze = nx.adj_matrix(T).A

    # let's draw the maze...
    maze_numpy = np.ones((rows*2+1, cols*2+1))
    for r in range(rows):
        for c in range(cols):
            maze_numpy[r*2 + 1, c*2 + 1] = 0
            node_num = get_node_number(r, c, rows, cols)
            # for everything adjacent - open up(?)
            conn_nodes = np.argwhere(maze[node_num, :] > 0).flatten()
            for conn_node in conn_nodes:
                diff_node = get_node_number_rev(conn_node, rows, cols)
                dx, dy = np.array(diff_node) - np.array([r, c])
                maze_numpy[r*2 + 1 + dx, c*2 + 1 + dy] = 0

    # let's randomly delete some walls
    for r in range(rows-1):
        for c in range(cols-1):
            counter = 0
            for dx, dy in [(-1, 0), (1, 0), (0, 1), (0, -1)]:
                r0, c0 = r*2+2+dx, c*2+2+dy
                if maze_numpy[r0, c0] == 0:
                    counter += 1
            # now do a random draw
            if np.random.random() > (4-counter)/4:
                maze_numpy[r*2+2, c*2+2] = 0

    return maze_numpy




class Maze(object):
    def __init__(self, maze = generate_maze(5, 5), goal=None, agent_pos=None):
        self.maze = maze
        self.goal = None
        self.agent_pos = None

        # up, down, left, right
        self.movement = [(-1, 0), (1, 0), (0, -1), (0, 1)]
    
    def reset(self):
        sample_pos = np.argwhere(self.maze == 0)
        agent, goal = np.random.choice(range(sample_pos.shape[0]), 2)
        self.agent_pos = tuple(list(sample_pos[agent]))
        self.goal = tuple(list(sample_pos[goal]))
    
    def step(self, action):
        dx, dy = self.movement(action)
        new_x, new_y = self.agent_pos[0] + dx, self.agent_pos[1] + dy
        if self.maze[new_x, new_y] != 1:
            self.agent_pos = (new_x, new_y)
        
        if self.agent_pos == self.goal:
            print("Final location found!")

    def set_agent(self, node):
        loc = get_node_number_rev(node, rows, cols)
        self.agent_pos = tuple(loc)
    
    def render(self, ascii=True):
        maze_mapper = {
            0: " ",
            1: "#",
            2: "@",
            3: "G"
        }
        maze = self.maze.copy()
        maze[self.goal] = 3
        maze[self.agent_pos] = 2        
        if ascii:
            print('\n'.join([''.join([maze_mapper[x] for x in r]) for r in maze.astype(int).tolist()]))
        return maze


if __name__ == "__main__":
    from os import system
    from time import sleep 

    # let's (re) solve this using networkx
    maze_env = Maze()
    maze_env.reset()
    maze = maze_env.maze
    rows, cols = maze.shape
    G = nx.Graph()
    G.add_nodes_from([x for x in range(rows*cols) if maze[tuple(get_node_number_rev(x, rows, cols))] != 1])
    # add edges when they're connected...
    for node in G.nodes:
        loc = get_node_number_rev(node, rows, cols)
        for dx, dy in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
            r, c = loc[0] + dx, loc[1] + dy
            if maze[r, c] != 1:
                conn_node = get_node_number(r, c, rows, cols)
                G.add_edge(node, conn_node)

    source = get_node_number(*list(maze_env.agent_pos) + [rows, cols])
    target = get_node_number(*list(maze_env.goal) + [rows, cols])
    for loc in  nx.shortest_path(G, source, target):
        maze_env.set_agent(loc)
        system('clear')
        maze_env.render()
        sleep(1)

```

The output of this maze is shown here:

![maze](img/maze/maze.gif)
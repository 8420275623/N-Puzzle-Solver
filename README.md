# N-Puzzle-Solver
# N Puzzle solver in Python for the HackerRank challenge, using IDA search, Manhattan distance and a time-bounded weighted search.


import sys

def solve(k, board):
    n = k * k
    goal = list(range(n))            # assumed goal: 0 1 2 ... (blank top-left)
    pos = [0] * n
    for i, v in enumerate(goal):
        pos[v] = i

    # solvability
    flat = [v for v in board if v]
    inv = sum(1 for i in range(len(flat)) for j in range(i + 1, len(flat)) if flat[i] > flat[j])
    br = board.index(0) // k
    gr = 0
    if k % 2 == 1:
        if inv % 2: return None
    else:
        if (inv + (br - gr)) % 2: return None

    row = [i // k for i in range(n)]
    col = [i % k for i in range(n)]

    def heuristic(b):
        h = 0
        for i, v in enumerate(b):
            if v:
                g = pos[v]
                h += abs(row[i] - row[g]) + abs(col[i] - col[g])
        # linear conflicts
        for r in range(k):
            line = [b[r * k + c] for c in range(k)]
            for a in range(k):
                va = line[a]
                if va and row[pos[va]] == r:
                    for c in range(a + 1, k):
                        vb = line[c]
                        if vb and row[pos[vb]] == r and col[pos[va]] > col[pos[vb]]:
                            h += 2
        for c in range(k):
            line = [b[r * k + c] for r in range(k)]
            for a in range(k):
                va = line[a]
                if va and col[pos[va]] == c:
                    for r2 in range(a + 1, k):
                        vb = line[r2]
                        if vb and col[pos[vb]] == c and row[pos[va]] > row[pos[vb]]:
                            h += 2
        return h

    b = board[:]
    path = []
    # blank moves: name, delta row, delta col
    dirs = [("UP", -1, 0), ("DOWN", 1, 0), ("LEFT", 0, -1), ("RIGHT", 0, 1)]
    opp = {"UP": "DOWN", "DOWN": "UP", "LEFT": "RIGHT", "RIGHT": "LEFT"}

    sys.setrecursionlimit(10000)

    def dfs(g, bound, z, last):
        h = heuristic(b)
        f = g + h
        if f > bound: return f
        if h == 0: return -1
        mn = 10**9
        zr, zc = divmod(z, k)
        for name, dr, dc in dirs:
            if last and opp[last] == name: continue
            nr, nc = zr + dr, zc + dc
            if 0 <= nr < k and 0 <= nc < k:
                nz = nr * k + nc
                b[z], b[nz] = b[nz], b[z]
                path.append(name)
                t = dfs(g + 1, bound, nz, name)
                if t == -1: return -1
                path.pop()
                b[z], b[nz] = b[nz], b[z]
                if t < mn: mn = t
        return mn

    bound = heuristic(b)
    z0 = b.index(0)
    while True:
        t = dfs(0, bound, z0, None)
        if t == -1: return path
        bound = t

def main():
    data = sys.stdin.read().split()
    k = int(data[0])
    board = list(map(int, data[1:1 + k * k]))
    res = solve(k, board)
    if res is None:
        print(-1)
    else:
        print(len(res))
        print("\n".join(res))

if __name__ == "__main__":
    main()

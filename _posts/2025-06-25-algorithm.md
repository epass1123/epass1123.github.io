---
title: "[백준/Node.js] 1018번: 체스판 다시 칠하기"
date: 2025-06-25
categories: [Algorithm, Baekjoon]
tags: [javascript, node.js, algorithm, brute-force]
---

게을러서 미루고 미뤘던 코딩테스트 공부를 다시 시작했습니다!

## 📝 문제

지민이는 자신의 저택에서 MN개의 단위 정사각형으로 나누어져 있는 M×N 크기의 보드를 찾았다. 어떤 정사각형은 검은색으로 칠해져 있고, 나머지는 흰색으로 칠해져 있다.

지민이는 이 보드를 잘라서 8×8 크기의 체스판으로 만들려고 한다.체스판은 검은색과 흰색이 번갈아서 칠해져 있어야 한다.

구체적으로, 각 칸이 검은색과 흰색 중 하나로 색칠되어 있고,변을 공유하는 두 개의 사각형은 다른 색으로 칠해져 있어야 한다.

따라서 이 정의를 따르면 체스판을 색칠하는 경우는 두 가지뿐이다. 하나는 맨 왼쪽 위 칸이 흰색인 경우, 하나는 검은색인 경우이다.
보드가 체스판처럼 칠해져 있다는 보장이 없어서, 지민이는 8×8 크기의 체스판으로 잘라낸 후에 몇 개의 정사각형을 다시 칠해야겠다고 생각했다. 당연히  크기는 아무데서나 골라도 된다.

지민이가 다시 칠해야 하는 정사각형의 최소 개수를 구하는 프로그램을 작성하시오.

### 입력

첫째 줄에 N과 M이 주어진다. N과 M은 8보다 크거나 같고, 50보다 작거나 같은 자연수이다.
둘째 줄부터 N개의 줄에는 보드의 각 행의 상태가 주어진다. B는 검은색이며, W는 흰색이다.

### 출력

첫째 줄에 지민이가 다시 칠해야 하는 정사각형 개수의 최솟값을 출력한다.
지민이는 M x N 크기의 보드를 가지고 있습니다. 이 보드는 검은색(B)과 흰색(W)으로 칠해져 있는데, 패턴이 일정하지 않습니다. 이 보드에서 8x8 크기의 체스판을 잘라내려고 합니다.

> [백준 1018번: 체스판 다시 칠하기 문제 링크](https://www.acmicpc.net/problem/1018)

## 💡 브루트포스

문제를 해결한 아이디어

> 1.  **모든 8x8 영역 탐색**: MN 보드에서 8x8 체스판을 잘라낼 수 있는 모든 시작 위치를 찾는다.
> 2.  **두 가지 체스판 패턴과 비교**: 각 8x8 영역에 대해, 'W'나 'B'로 시작하는 정답 체스판 두 가지 경우를 모두 고려한다.
두 체스판과 비교하여 다시 칠해야 할 칸의 개수를 구한다.
{: .prompt-tip }


## 💻 코드 구현 (JavaScript)

이제 위 전략을 바탕으로 작성한 전체 코드입니다.


```javascript


const fs = require('fs');
const input = fs.readFileSync('/dev/stdin').toString().trim().split('\n');

let answer = 64;
const [M, N] = input.shift().trim().split(' ');
const board = input.map(row => row.trim());

const bRow = ['BWBWBWBW', 'WBWBWBWB', 'BWBWBWBW', 'WBWBWBWB', 'BWBWBWBW', 'WBWBWBWB', 'BWBWBWBW', 'WBWBWBWB'];
const wRow = ['WBWBWBWB', 'BWBWBWBW', 'WBWBWBWB', 'BWBWBWBW', 'WBWBWBWB', 'BWBWBWBW', 'WBWBWBWB', 'BWBWBWBW'];

// 모든 8x8 영역 탐색 (브루트포스)
for (let startRow = 0; startRow <= M - 8; startRow++) {
  for (let startCol = 0; startCol <= N - 8; startCol++) {
    let paintBlackStart = 0; // 'B'로 시작하는 체스판과 비교 시 다시 칠할 개수
    let paintWhiteStart = 0; // 'W'로 시작하는 체스판과 비교 시 다시 칠할 개수

    // 현재 시작 위치에서 비교
    for (let i = 0; i < 8; i++) {
      for (let j = 0; j < 8; j++) {
        // 'B'로 시작하는 패턴과 비교
        if (board[startRow + i][startCol + j] !== bRow[i][j]) {
          paintBlackStart++;
        }

        // 'W'로 시작하는 패턴과 비교
        if (board[startRow + i][startCol + j] !== wRow[i][j]) {
          paintWhiteStart++;
        }
      }
    }
    
    //최소값 갱신
    answer = Math.min(answer, Math.min(paintBlackStart, paintWhiteStart));
  }
}

console.log(answer);
```

오랜만에 할려니까 완전탐색도 힘들어서 결국 여기저기 참고해서 풀어버렸습니다..  
완전탐색 다른 문제는 혼자할 수 있기를
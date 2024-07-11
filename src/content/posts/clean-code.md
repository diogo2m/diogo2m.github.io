---
title: "Writing a nice code"
description: "A guide to writing good code using Clean Code principles."
date: 2024-07-11
image: "/images/posts/coding/clean-code.jpg"
categories: ["materials"]
authors: ["Diogo Monteiro"]
tags: ["clean code", "java"]
draft: false
---

I'm going to introduce you to my personal method of code development. This is not a definitive tutorial, but it can help programmers find a way to create good code while minimizing refactoring time. In this material, I'll be using Java as the programming language, but the principles can be applied to any programming language.

### 1. Think about your problem.
What do you need to solve? How can it be made more efficient? Take an overhead view of your problem, noting everything relevant that comes to mind. Also, try to decompose your problem into many smaller problems; this increases your code's legibility and facilitates development.

### 2. Write your main function
Yes, you read that right. As counterintuitive as it may seem, creating the main function first can enhance your code's legibility and algorithmic logic.

### 3. Function organization
Following Clean Code principles (detailed in "Clean Code: A Handbook of Agile Software Craftsmanship" by Robert C. Martin), creating small and objective functions can potentially increase your code's legibility.

### 4. Comments
Comments are extremely important for the quick understanding and refactoring of your code. As your functions get more complex, you will need to write more comments to properly understand your functionality.

### 5. Naming
Naming can quickly help you understand your code without using any comments. Great function and variable names can reduce or even avoid the need for comments. Also, they can increase your code's legibility and algorithmic logic.

All these established concepts can be applied recursively to your code. For example, if you are working with multiple classes, you can apply the same principles to each class.

## Practical example

Let's say you're building a tic-tac-toe game.

To play a tic-tac-toe game, you need to create a board with 9 cells. This board have several functionalities such as checking if a cell is empty, checking if a cell is full, and checking if a cell is a winning one. Given that, you can create a class called Board.

```java
public class Board {
    char[][] board;

    // Constructors are used to initialize the attributes of the class.
    public Board() {
        board = new char[3][3];
    }

    public addMark(int row, int column, char value) {
        board[row][column] = value;
    }

    public boolean isEmpty(int row, int column) {
       ...
    }

    public boolean isFull(int row, int column) {
       ...
    }
```

Also, we need to verify and check if a mark is a winning one. Given that our game has continuous insertions, we don't need to verify if there is a winner in cells that we didn't change. We can just check if an inserted cell formed a line, a column, or a diagonal.

```java 
    public boolean isWinning(int row, int column) {
       ...
    }
}
```

Using the above functions, we can create a class called TicTacToe. This class will have a main function that will ask repeatedly the users to play the game while are not a winner (or the game is over).

```java
public class TicTacToe {
    public static void main(String[] args) {
        Board board = new Board();

        while (board.isGameOver() == false){
            if (board.timeToPlay() == 'x'){
                // Ask for X players to play (creating a function or using System.out and System.in directly)
                // Verify if the coordinates are valid (using board.isFull)
                // Set the cell to X (using board.addMark)
                // Check if the cell is a winning one (board.isWinning), if true, announce X's victory and finish execution.
            }
            if (board.timeToPlay() == 'o'){
                // Ask for O players to play (creating a function or using System.out and System.in directly)
                // Verify if the coordinates are valid (using board.isFull)
                // Set the cell to O (using board.addMark)
                // Check if the cell is a winning one (board.isWinning), if true, announce O's victory and finish execution.
            }
        }

        // Announce game over and finish execution.
```
Therefore, with just a little bit of thought, we can create a class that can be used to play tic-tac-toe games. Moreover, these functions follow the best practices of Clean Code and can be utilized by other programmers to produce improved code.

This technique is also very useful when you use an artificial intelligence to solve a problem. Breaking down the problem into smaller problems can enhance AI certainty and accuracy by reducing the complexity of the problem.

## Conclusion
I hope this material has helped you understand the importance of writing good code. I'm confident you'll find these principles useful in your future projects. If you have any questions or comments, feel free to reach out to me on LinkedIn, Instagram, or via email.

Happy coding!

#include <iostream>
#include <string>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::vector;
using std::string;

class TicTacToe { 
private:
    vector<vector<string>> board;
    string currentPlayer;
    int win_count_cells;

public:
  TicTacToe(int size) : win_count_cells(size) {
    board = vector<vector<string>>(size, vector<string>(size, " "));
  }

  void printBoard() {
    cout << "  ";
    for (int j = 0; j < board.size(); ++j) {
      cout << j << ' ';
    }
    cout << endl;
    for (int i = 0; i < board.size(); ++i) {
      cout << i << '|';
      for (int j = 0; j < board.size(); ++j) {
        cout << board[i][j];
        if (j < board.size() - 1){
          cout << '|';
        }
      }
      cout << '|';
      cout << endl;
      if (i < board.size() - 1) {
        cout << " +";
        for (int j = 0; j < board.size(); ++j) {
          cout << "-";
          if (j < board.size() - 1)
            cout << '+';
        }
        cout << '+';
        cout << endl;
      }
    }
  }

  bool placeMarker(int row, int col) {
    if (row >= 0 && row < board.size() && col >= 0 && col < board.size() &&
        board[row][col] == " ") {
      board[row][col] = currentPlayer;
      return true;
    }
    return false;
  }

  bool checkWin() {
    int size = board.size();

    for (int i = 0; i < size; ++i) {
      for (int j = 0; j <= size - win_count_cells; ++j) {
        if (checkLine(i, j, 0, 1) || checkLine(j, i, 1, 0))
          return true;
      }
    }

    for (int i = 0; i <= size - win_count_cells; ++i) {
      for (int j = 0; j <= size - win_count_cells; ++j) {
        if (checkLine(i, j, 1, 1))
          return true;
        if (checkLine(i, j + win_count_cells - 1, 1, -1))
          return true;
      }
    }

    return false;
  }
  bool checkLine(int startRow, int startCol, int deltaRow, int deltaCol) {
    string first = board[startRow][startCol];
    if (first == " ")
      return false;

    for (int i = 1; i < win_count_cells; ++i) {
      if (board[startRow + i * deltaRow][startCol + i * deltaCol] != first) {
        return false;
      }
    }
    return true;
  }
  void switchPlayer() {
    currentPlayer = (currentPlayer == "X") ? "O" : "X"; 
  }
  void play() {
    int row, col;
    int turns = 0;
    cout << "Выберите, кто ходит первым (X или O): ";
    cin >> currentPlayer;
    checker(currentPlayer);
    while (turns < board.size() * board.size()) {
      cout << "\033[2J";
      cout << "\033[H";
      printBoard();
      cout << "Игрок " << currentPlayer
           << ", введите строку и столбец (например, 1 1): ";
      cin >> row >> col;
      while (true) {
        if (placeMarker(row, col)) {
          if (checkWin()) {
            cout << "\033[2J";
            cout << "\033[H";
            printBoard();
            cout << "Игрок " << currentPlayer << " выиграл!\n";
            return;
          }
          switchPlayer();
          turns++;
          break;
        } else {
          cout << "\x1b[1A" << "\x1b[2K";
          cout << "Некорректный ход. Попробуйте снова.\n";
          cin >> row >> col;
        }
      }
    }
    cout << "\033[2J";
    cout << "\033[H";
    printBoard();
    cout << "Игра завершилась вничью!\n";
  }
  void checker(string currentPLayer) {
    if (currentPlayer == "X" || currentPlayer == "x") {
      currentPlayer = 'X';
    } else if (currentPlayer == "O" || currentPlayer == "o") {
      currentPlayer = 'O';
    }
    while (currentPlayer != "X" && currentPlayer != "x" &&
           currentPlayer != "O" && currentPlayer != "o") {
      cout << "\x1b[1A" << "\x1b[2K";
      cout << "Некорректный ввод. Введите X или O: ";
      cin >> currentPlayer;
      checker(currentPlayer);
    }
  }
};
int starter() {
  int size;
  cout << "\nВыберите уровень сложности:\nЛегко(3),\nСредне(4),\nСложно(5),\n"
    "Экстрим(6),\nCustom(введите свой размер поля не больше 10);\n(Вводите только "
    "числа): ";
  cin >> size;
  while (size < 3 || size > 10) {
    cout << "\x1b[1A" << "\x1b[2K";
    cout << "Некорректный размер. Пожалуйста, введите число от 3 до 10: ";
    cin >> size;
  }
  return size;
}
void continuous(string continue_game) {
  int size;
  while ((continue_game != "да" && continue_game != "Да") && (continue_game != "нет" && continue_game != "Нет")) {
    cout << "\x1b[1A" << "\x1b[2K";
    cout << "Некорректный ввод. Пожалуйста, введите да или нет.\n";
    cin >> continue_game;
  }
  if (continue_game == "да" || continue_game == "Да") {
    cout << "\033[2J";
    cout << "\033[H";
    size = starter();
    TicTacToe game(size);
    game.play();
    cout << "Продолжить игру?(Да или Нет)\n";
    cin >> continue_game;
    continuous(continue_game);
  } else if (continue_game == "нет" || continue_game == "Нет"){
    cout << "Конец игры.\n";
    return;
  }
}
int main() {
  int size = starter();
  TicTacToe game(size);
  game.play();
  cout << "Продолжить игру?(Да или Нет)\n";
  string continue_game; 
  cin >> continue_game;
  continuous(continue_game);
}
#include <iostream>
#include <string>
#include <vector>
#include <limits>

using std::cin;
using std::cout;
using std::endl;
using std::vector;
using std::string;

class TicTacToe {
private:
    // Игровое поле
    vector<vector<string>> board;
    // Текущий игрок (X или O)
    string currentPlayer;
    // Количество ячеек для победы
    int win_count_cells;
    // Флаг, указывающий, играет ли кто-то против бота
    bool vsBot = false;
    // Идентификаторы игроков
    string humanPlayer;
    string botPlayer;

    // Проверка линии (горизонтальной, вертикальной или диагональной)
    bool checkLine(int startRow, int startCol, int deltaRow, int deltaCol) {
        string first = board[startRow][startCol];
        // Если ячейка пустая, линия не найдена
        if (first == " ") return false;

        // Проверяем остальные ячейки в указанном направлении
        for (int i = 1; i < win_count_cells; ++i) {
            if (board[startRow + i * deltaRow][startCol + i * deltaCol] != first) {
                return false;
            }
        }
        return true;
    }

    // Проверка на победу для игрока
    bool checkWin(const string& player) {
        int size = board.size();
        // Проверка горизонтальных и вертикальных линий
        for (int i = 0; i < size; ++i) {
            for (int j = 0; j <= size - win_count_cells; ++j) {
                if ((checkLine(i, j, 0, 1) && board[i][j] == player) || // Горизонтальная проверка
                    (checkLine(j, i, 1, 0) && board[j][i] == player))   // Вертикальная проверка
                    return true;
            }
        }

        // Проверка диагональных линий
        for (int i = 0; i <= size - win_count_cells; ++i) {
            for (int j = 0; j <= size - win_count_cells; ++j) {
                if ((checkLine(i, j, 1, 1) && board[i][j] == player) || // Диагональная "\"
                    (checkLine(i, j + win_count_cells - 1, 1, -1) && board[i][j + win_count_cells - 1] == player)) // Диагональная "/"
                    return true;
            }
        }
        return false; // Победа не найдена
    }

    // Алгоритм минимакса для определения лучшего хода для бота
    int minimax(bool isMaximizing) {
        if (checkWin(botPlayer)) return 1; // Бот выигрывает
        if (checkWin(humanPlayer)) return -1; // Человек выигрывает

        bool isFull = true; // Проверка, заполнено ли поле
        for (auto& row : board) {
            for (auto& cell : row) {
                if (cell == " ") {
                    isFull = false; // Если найдена пустая ячейка, поле не заполнено
                    break;
                }
            }
            if (!isFull) break;
        }
        if (isFull) return 0; // Ничья

        int bestScore = isMaximizing ? -1000 : 1000; // Инициализация наивысшего (или наименьшего) результата
        for (int i = 0; i < board.size(); ++i) {
            for (int j = 0; j < board.size(); ++j) {
                if (board[i][j] == " ") {
                    // Пробуем поставить ход
                    board[i][j] = isMaximizing ? botPlayer : humanPlayer;
                    int score = minimax(!isMaximizing); // Рекурсивный вызов
                    board[i][j] = " "; // Возвращаем назад

                    // Обновляем наилучший результат
                    if (isMaximizing) {
                        if (score > bestScore) bestScore = score;
                    } else {
                        if (score < bestScore) bestScore = score;
                    }
                }
            }
        }
        return bestScore; // Возвращаем лучший результат
    }

    // Ход бота
    void botMove() {
        int bestScore = -1000; // Начальная оценка для минимакса
        int bestRow = -1, bestCol = -1;

        for (int i = 0; i < board.size(); ++i) {
            for (int j = 0; j < board.size(); ++j) {
                if (board[i][j] == " ") {
                    board[i][j] = botPlayer; // Попробовать ход бота
                    int score = minimax(false); // Запускаем минимакс
                    board[i][j] = " "; // Возвращаем назад

                    if (score > bestScore) { // Сохраняем лучшее место
                        bestScore = score;
                        bestRow = i;
                        bestCol = j;
                    }
                }
            }
        }
        placeMarker(bestRow, bestCol); // Делаем лучший ход
    }

public:
    // Конструктор
    TicTacToe(int size, bool useBot = false) : win_count_cells(size), vsBot(useBot) {
        board = vector<vector<string>>(size, vector<string>(size, " ")); // Инициализация поля
    }

    // Печать игрового поля
    void printBoard() {
        cout << "  ";
        for (int i = 0; i < board.size(); ++i) {
            cout << i << ' '; // Индексы столбцов
        }
        cout << endl;

        for (int i = 0; i < board.size(); ++i) {
            cout << i << '|';
            for (int j = 0; j < board.size(); ++j) {
                cout << board[i][j]; // Печатаем ячейки
                if (j < board.size() - 1) {
                    cout << '|';
                }
            }
            cout << '|' << endl;
            if (i < board.size() - 1) {
                cout << " +";
                for (int j = 0; j < board.size(); ++j) {
                    cout << "-";
                    if (j < board.size() - 1)
                        cout << '+';
                }
                cout << '+' << endl;
            }
        }
    }

    // Размещение маркера на игровом поле
    bool placeMarker(int row, int col) {
        if (row >= 0 && row < board.size() && col >= 0 && col < board.size() &&
            board[row][col] == " ") { // Проверяем корректность ввода
            board[row][col] = currentPlayer; // Ставим маркер текущего игрока
            return true;
        }
        return false; // Если ход недействителен
    }

    // Смена текущего игрока
    void switchPlayer() {
        currentPlayer = (currentPlayer == "X") ? "O" : "X"; 
    }

    // Проверка корректности ввода игрока
    void checker(string inputPlayer) {
        if (inputPlayer == "X" || inputPlayer == "x") {
            currentPlayer = "X";
            if (vsBot) {
                humanPlayer = "X"; // Человек - Х
                botPlayer = "O";   // Бот - O
            }
        } else if (inputPlayer == "O" || inputPlayer == "o") {
            currentPlayer = "O";
            if (vsBot) {
                humanPlayer = "O"; // Человек - O
                botPlayer = "X";   // Бот - X
            }
        }
        // Повторный ввод при ошибке
        while (currentPlayer != "X" && currentPlayer != "x" &&
               currentPlayer != "O" && currentPlayer != "o") {
            cout << "\x1b[1A" << "\x1b[2K" << "Некорректный ввод. Введите X или O: ";
            cin >> inputPlayer;
            checker(inputPlayer);
        }
    }

    // Основная игровая логика
    void play() {
        int row, col;
        int turns = 0; // Счетчик ходов
        cout << "Выберите, кто ходит первым (X или O): ";
        cin >> currentPlayer;
        checker(currentPlayer);
        
        while (turns < board.size() * board.size()) {
            cout << "\033[2J" << "\033[H"; // Очистка экрана
            printBoard();

            // Ход бота
            if (currentPlayer == botPlayer) {
                botMove(); // Бот делает ход
                if (checkWin(currentPlayer)) {
                    cout << "\033[2J\033[H"; // Очистка экрана
                    printBoard();
                    cout << "Бот " << currentPlayer << " выиграл!\n";
                    return;
                }
                switchPlayer(); // Смена игрока
                turns++;
                continue;
            }
 
            // Ход игрока
            cout << "Игрок " << currentPlayer << ", введите строку и столбец (например, 1 1): ";
            cin >> row >> col;
            while (true) {
                if (placeMarker(row, col)) { // Если ход успешен
                    if (checkWin(currentPlayer)) {
                        cout << "\033[2J" << "\033[H"; // Очистка экрана
                        printBoard();
                        cout << "Игрок " << currentPlayer << " выиграл!\n";
                        return;
                    }
                    switchPlayer(); // Смена игрока
                    turns++;
                    break;
                } else {
                    cout << "\x1b[1A" << "\x1b[2K";
                    cout << "Некорректный ход. Попробуйте снова: ";
                    cin >> row >> col; // Повторный ввод
                }
            }
        }
        cout << "\033[2J" << "\033[H"; // Очистка экрана
        printBoard();
        cout << "Игра завершилась вничью!\n"; // Если ничья
    }
};

// Функция для старта игры
int starter(bool& useBot) {
    int size;
    cout << "\nИграть против бота? (да/нет): ";
    string answer;
    cin >> answer;
    // Проверка корректного ввода
    while (answer != "Да" && answer != "да" && answer != "Нет" && answer != "нет") {
        cout << "\x1b[1A" << "\x1b[2K" << "Некорректный ввод. Введите да или нет: ";
        cin >> answer;
    }
    useBot = (answer == "да" || answer == "Да");
    // Если играем против бота, фиксируем размер поля 3
    if (useBot) {
        size = 3;
    } else {
        // Ввод размера поля
        cout << "Выберите размер поля (3-10): ";
        cin >> size;
        // Проверка корректного диапазона поля
        while (size < 3 || size > 10) {
            if (cin.fail()) {
                cin.clear();
                cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
            }
            cout << "\x1b[1A" << "\x1b[2K" << "Некорректный размер. Введите число от 3 до 10: ";
            cin >> size;
        }
    }
    return size; // Возвращаем размер поля
}

int main() {
    bool vsBot; // Переменная для определения игры против бота
    int size = starter(vsBot); // Запуск функции для выбора параметров игры
    TicTacToe game(size, vsBot); // Инициализация игры
    game.play(); // Начало игры
    cout << "Продолжить игру? (да/нет)\n";
    string continue_game;
    cin >> continue_game;
    while (true) {
        // Проверяем, хочет ли игрок продолжить игру
        if (continue_game == "Да" || continue_game == "да") {
            size = starter(vsBot);
            TicTacToe newGame(size, vsBot); // Создаем новую игру
            newGame.play();
            cout << "Продолжить игру? (да/нет)\n";
            cin >> continue_game;
        } else if (continue_game == "Нет" || continue_game == "нет") {
            cout << "Конец игры.\n"; // Завершение игры
            return 0;
        } else {
            cout << "\x1b[1A\x1b[2K";
            cout << "Некорректный ввод. Введите да или нет: ";
            cin >> continue_game;
        }
    }
}
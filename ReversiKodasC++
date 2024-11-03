#include <iostream>
#include <string>
#include <iomanip>
#include <Windows.h>
#include <map>
#include <deque>
#include <cctype>
#include <vector>

#define Deletion "\33[2K\r"
#define PreviousLine "\033[A\r"

using namespace std;

void RowDeletion(int rowCount);

class Board {
public:
    map<pair<char, short>, char> boardInfo;
    map<char, short> columnNumberByLetter;
    char columnLetters[8] = { 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H' };
    Board() {
        for (int i = 0; i < 8; i++) {
            for (int j = 0; j < 8; j++) {
                boardInfo[{columnLetters[i], j + 1}] = 'O';
            }
            columnNumberByLetter[columnLetters[i]] = i + 1;
        }
    }
    void PrintBoard() {
        cout << setw(63) << "BLACK DISKS PLAYER TURN\n\n";
        cout << setw(65) << " A   B   C   D   E   F   G   H\n";
        for (int i = 0; i < 8; i++) {
            Row();
            cout << setw(32) << i + 1;
            Columns(i);
        }
        Row();
    }
    void UpdateBoard(char letter, short number, char player) {
        wstring color = L"";

        boardInfo[{letter, number}] = player;

        wcout << L"\x1b[0;39H";
        if (player == 'B') {
            color = L"\033[42;30m•\033[42;30m";
            wcout << L"\033[0mWHITE DISKS PLAYER TURN\033[0m";
        }
        else if (player == 'W') {
            color = L"\033[42;37m•\033[42;30m";
            wcout << L"\033[0mBLACK DISKS PLAYER TURN\033[0m";
        }
        wcout << L"\x1b[" << number * 2 + 3 << ";" << 32 + columnNumberByLetter[letter] * 4 << "H";
        wcout << color;
        wcout << L"\x1b[21;0H\033[0m";
    }
    void Reversing(vector<pair<char, short>> flanks, char player) {
        for (auto tile : flanks) {
            UpdateBoard(tile.first, tile.second, player);
        }
    }
    bool IsActionInCenter() {
        if (boardInfo[{'D', 4}] == 'O' || boardInfo[{'D', 5}] == 'O'
            || boardInfo[{'E', 4}] == 'O' || boardInfo[{'E', 5}] == 'O') return true;
        return false;
    }
    void GameEnding() {
        CalculatePoints();
        cout << "\n\n\nGAME ENDED\n";
        if (whitePoints > blackPoints) cout << "Winner is White disks player!\n";
        else if (whitePoints < blackPoints) cout << "Winner is Black disks player!\n";
        cout << "Black disks player points: " << blackPoints << endl;
        cout << "White disks player points: " << whitePoints;
    }
    void CalculatePoints() {
        for (auto tile : boardInfo) {
            if (tile.second == 'W') whitePoints++;
            else if (tile.second == 'B') blackPoints++;
        }
    }
private:
    short whitePoints = 0;
    short blackPoints = 0;
    void Row() {  
        cout << setw(79) << "\033[42;30m---------------------------------\033[0m\n";
    }
    void Columns(short row) {
        cout << setw(10) << "\033[42;30m|";
        for (int i = 0; i < 8; i++) {
            cout << "   |";
       }
        cout << "\033[0m\n";
    }
};

class PlayerBase {
public:
    Board gameBoard;
    map<pair<char, short>, char> boardInfo;
    bool isActionInCenter = true;
    virtual void UpdateDisks(Board& board) = 0;
    void Input(char player) {
        string userInput = "";
        char columnLetter = ' ';
        short rowNumber = 0;

        while (true) {
            getline(cin, userInput);
            if (!userInput.empty() && userInput.length() >= 2) {
                columnLetter = userInput[0];
                if (isdigit(userInput[1])) rowNumber = stoi(userInput.substr(1, 1));
                if (!isdigit(columnLetter) && rowNumber > 0) break;
            }
            RowDeletion(1);
        }

        if (rowNumber < 1 || rowNumber > 8
            || toupper(columnLetter) > 'H'
            || toupper(columnLetter) < 'A'
            || !IsInputValid(player, toupper(columnLetter), rowNumber)) {
            RowDeletion(1);
            Input(player);
        }
        else {
            RowDeletion(1);
            this->columnLetter = toupper(columnLetter);
            this->rowNumber = rowNumber;
            if (flanks.size() > 0) {
                Reversing(gameBoard, player);
                flanks.clear();
            }
        }
    }
    void GetGameBoardInfo(Board& board) {
        this->gameBoard = board;
        this->boardInfo = board.boardInfo;
        this->isActionInCenter = board.IsActionInCenter();
    }
    bool IsGameEnding(char opponent) {
        if (isActionInCenter) return false;
        bool canFlank = false;

        for (auto tile : boardInfo) {
            if (tile.second == 'O') {
                for (auto direction : directions) {
                    canFlank = CanFlankOpponent(direction[0], direction[1], opponent, GetColumnNumber(tile.first.first), tile.first.second);
                    currentFlanks.clear();
                    flanks.clear();
                    if (canFlank) { 
                        return false; 
                    }
                }
            }
        }
        return true;
    }
protected:
    char columnLetter = ' ';
    short rowNumber = 0;
    short directions[8][2] = {
         {-1, 0}, {1, 0}, {0, -1}, {0, 1},
         {-1, -1}, {-1, 1}, {1, -1}, {1, 1} };
    vector<pair<char, short>> flanks;
    vector<pair<char, short>> currentFlanks;

    bool IsInputValid(char player, char colLetter, short rowNum) {
        if (isActionInCenter) {
            if ((colLetter == 'D' && rowNum == 4 && boardInfo[{'D', 4}] == 'O')
                || (colLetter == 'D' && rowNum == 5 && boardInfo[{'D', 5}] == 'O')
                || (colLetter == 'E' && rowNum == 4 && boardInfo[{'E', 4}] == 'O')
                || (colLetter == 'E' && rowNum == 5) && boardInfo[{'E', 5}] == 'O') return true;
        }
        else {
            if (boardInfo[{colLetter, rowNum}] == 'O') {
                char opponent = (player == 'B') ? 'W' : 'B';
                short currentColNum = GetColumnNumber(colLetter);

                for (auto direction : directions) {
                    short x = direction[0];
                    short y = direction[1];

                    if (boardInfo[{GetColumnLetterByNumber(currentColNum + x), 
                        rowNum + y}] == opponent) {   
                        currentFlanks.clear();
                        if (CanFlankOpponent(x, y, opponent, currentColNum, rowNum)) {
                            if (currentFlanks.size() > 0) {
                                flanks.insert(flanks.end(), currentFlanks.begin(), currentFlanks.end());
                            }
                       }
                    }
                }
                if (flanks.size() > 0) return true;
            }
        }
        return false;
    }
    bool CanFlankOpponent(short flankingDirX, short flankingDirY, char opponent, short currentX, short currentY) {
        short dirX = currentX + flankingDirX;
        short dirY = currentY + flankingDirY;
        
        currentFlanks.push_back({GetColumnLetterByNumber(dirX), dirY});
        if (boardInfo[{GetColumnLetterByNumber(dirX), dirY}] == NULL
            || boardInfo[{GetColumnLetterByNumber(dirX), dirY}] == 'O') {
            currentFlanks.clear();
            return false;
        }
        else if (boardInfo[{GetColumnLetterByNumber(dirX), dirY}] == opponent) {
            CanFlankOpponent(flankingDirX, flankingDirY, opponent, dirX, dirY);
        }
        else {
            currentFlanks.pop_back();
            return true;
        }
    }
    void Reversing(Board &board, char player) {
        board.Reversing(flanks, player);
    }
    short GetColumnNumber(char letter) {
        return gameBoard.columnNumberByLetter[letter];
    }
    char GetColumnLetterByNumber(short indx) {
        return gameBoard.columnLetters[indx - 1];
    }
};

class PlayerBlack : public PlayerBase {
public:
    PlayerBlack() {
        columnLetter = ' ';
        rowNumber = 0;
    }
    void UpdateDisks(Board &board) {
        board = this->gameBoard;
        board.UpdateBoard(columnLetter, rowNumber, 'B');
    }
};

class PlayerWhite : public PlayerBase {
public:
    PlayerWhite() {
        columnLetter = ' ';
        rowNumber = 0;
    }
    void UpdateDisks(Board &board) {
        board = this->gameBoard;
        board.UpdateBoard(columnLetter, rowNumber, 'W');
    }
};

int main()
{
    SetConsoleOutputCP(CP_UTF8);
    wcout.imbue(std::locale("en_US.UTF-8"));

    bool isFinished = false;
    char whichPlayersTurn = 'B';
    char opponent = 'W';

    deque<unique_ptr<PlayerBase>> Players;
    Board board;

    Players.push_back(make_unique<PlayerBlack>());
    Players.push_back(make_unique<PlayerWhite>());

    board.PrintBoard();

    while (!isFinished) {
        for (auto &player : Players) {
            if (dynamic_cast<PlayerBlack*>(player.get())) {
                whichPlayersTurn = 'B';
                opponent = 'W';
            }
            else if (dynamic_cast<PlayerWhite*>(player.get())) {
                whichPlayersTurn = 'W';
                opponent = 'B';
            }
            player->GetGameBoardInfo(board);
            if (player->IsGameEnding(opponent)) {
                board.GameEnding();
                isFinished = true;
                break;
            }
            player->Input(whichPlayersTurn);
            player->UpdateDisks(board);
        }
    }
    return 0;
}

void RowDeletion(int rowCount) {
    for (int i = 0; i < rowCount; i++) {
        cout << PreviousLine;
        cout << Deletion;
    }
}

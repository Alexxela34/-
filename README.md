#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <map>
#include <random>
#include <chrono>

using namespace std;

class FastRand {
    mt19937 rng;
public:
    FastRand() : rng(chrono::steady_clock::now().time_since_epoch().count()) {}
    int next(int min, int max) {
        uniform_int_distribution<int> dist(min, max);
        return dist(rng);
    }
};

class Dictionary {
    map<int, vector<string>> dict;
    const vector<int> lvls{ 4, 7, 10 };

public:
    void addWord(const string& word) {
        int lvl = 0;
        while (lvl < (int)lvls.size() && (int)word.size() > lvls[lvl]) ++lvl;
        dict[lvl].push_back(word);
    }

    string random(int lvl) {
        if (dict.find(lvl) == dict.end() || dict[lvl].empty()) return "";
        FastRand frand;
        const auto& words = dict[lvl];
        return words[frand.next(0, (int)words.size() - 1)];
    }
};

class Gallow {
    vector<string> gallow;
    vector<char> field;
    unsigned tries = 9;
    string word;

    void loadGallow() {
        ifstream file("draw.txt");
        gallow.clear();
        for (int i = 0; i < 11; ++i) {
            string block, line;
            for (int j = 0; j < 10; ++j) {
                if (getline(file, line)) block += line + "\n";
            }
            gallow.push_back(block);
        }
    }

public:
    Gallow(const string& w) : word(w), field(w.size(), '_') {
        loadGallow();
    }

    size_t getTries() const { return tries; }
    size_t remainder() const {
        return count(field.begin(), field.end(), '_');
    }

    string draw() const {
        return gallow[9 - tries];
    }

    vector<char>& getField() {
        return field;
    }

    void substitute(char let) {
        bool found = false;
        for (size_t i = 0; i < word.size(); ++i) {
            if (word[i] == let && field[i] == '_') {
                field[i] = let;
                found = true;
            }
        }
        if (!found) --tries;
    }

    string getWord() const {
        return word;
    }
};

ostream& operator<<(ostream& out, const vector<char>& field) {
    for (char c : field) out << c << ' ';
    return out;
}

int main() {
    ifstream file("words.txt");
    Dictionary dict;
    string word;
    while (file >> word) dict.addWord(word);

    int lvl;
    cout << "Choose difficulty level (0-2): ";
    cin >> lvl;

    string chosenWord = dict.random(lvl);
    if (chosenWord.empty()) {
        cout << "No words available for this level." << endl;
        return 1;
    }

    Gallow game(chosenWord);
    char letter;

    while (game.getTries() > 0 && game.remainder() > 0) {
        system("cls");
        cout << game.draw() << endl;
        cout << game.getField() << endl;
        cout << "Select a letter: ";
        cin >> letter;
        game.substitute(letter);
    }

    system("cls");
    cout << game.draw() << endl;
    cout << game.getField() << endl;
    if (game.remainder() == 0)
        cout << "\nYou won!" << endl;
    else
        cout << "\nLOOSER!\nThe word was: " << game.getWord() << endl;

    return 0;
}

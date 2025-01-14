# Cricket-Score-Sheet
#include "raylib.h"
#include <vector>
#include <string>
#include <sstream>
using namespace std;

// Player class
class Player {
private:
    string name;
    int runs;
    int ballsFaced;
    int status; // 0: not out, 1: bowled, etc.

public:
    Player(string name = "Player") : name(name), runs(0), ballsFaced(0), status(0) {}

    string getName() const { return name; }
    int getRuns() const { return runs; }
    int getBallsFaced() const { return ballsFaced; }
    string getStatus() const {
        switch (status) {
            case 0: return "not out";
            case 1: return "bowled";
            case 2: return "caught";
            case 3: return "run out";
            default: return "unknown";
        }
    }

    void addRuns(int run) { runs += run; }
    void incrementBalls() { ballsFaced++; }
    void setStatus(int newStatus) { status = newStatus; }
};

// CricketTeam class
class CricketTeam {
private:
    string teamName;
    vector<Player> players;
    int extraRuns;

public:
    CricketTeam(string name = "Team") : teamName(name), extraRuns(0) {}

    void initializeTeam(const vector<string>& playerNames) {
        players.clear();
        for (const auto& name : playerNames) {
            players.emplace_back(name);
        }
    }

    string getTeamName() const { return teamName; }
    int getTotalScore() const {
        int totalScore = extraRuns;
        for (const auto& player : players) totalScore += player.getRuns();
        return totalScore;
    }
    int getExtraRuns() const { return extraRuns; }
    const Player& getPlayer(int index) const { return players.at(index); }
    Player& getPlayer(int index) { return players.at(index); }
    void addExtraRuns(int runs) { extraRuns += runs; }
};

// Game Manager
class CricketGame {
private:
    CricketTeam team;
    int batsman1Index, batsman2Index, ballCount, overs, maxOvers, wickets;

public:
    CricketGame() : batsman1Index(0), batsman2Index(1), ballCount(0), overs(0), wickets(0), maxOvers(10) {}

    void initialize(const string& teamName, const vector<string>& playerNames, int oversLimit) {
        team = CricketTeam(teamName);
        team.initializeTeam(playerNames);
        maxOvers = oversLimit;
    }

    string getScoreDetails() const {
        stringstream ss;
        ss << "Team: " << team.getTeamName() << "\n";
        ss << "Overs: " << overs << "." << ballCount << "\n";
        ss << "Wickets: " << wickets << "\n";
        ss << "Total Score: " << team.getTotalScore() << "\n\n";

        for (size_t i = 0; i < 11; i++) {
            const Player& player = team.getPlayer(i);
            ss << player.getName() << " " << player.getStatus() << " " << player.getRuns()
               << "(" << player.getBallsFaced() << ")\n";
        }
        return ss.str();
    }

    void registerDotBall() {
        ballCount++;
        team.getPlayer(batsman1Index).incrementBalls();
        if (ballCount == 6) {
            ballCount = 0;
            overs++;
        }
    }

    void registerRuns(int runs) {
        ballCount++;
        team.getPlayer(batsman1Index).addRuns(runs);
        if (runs % 2 == 1) {
            swap(batsman1Index, batsman2Index);
        }
        if (ballCount == 6) {
            ballCount = 0;
            overs++;
        }
    }

    void registerExtras(int runs) { team.addExtraRuns(runs); }

    void registerWicket() {
        team.getPlayer(batsman1Index).setStatus(1); // Mark as bowled
        wickets++;
        batsman1Index = (wickets + 1 < 11) ? wickets + 1 : batsman1Index; // Assign next player
    }
};

int main() {
    // Initialize Raylib
    InitWindow(800, 600, "Cricket Score Manager");
    SetTargetFPS(60);

    // Initialize game
    CricketGame game;
    game.initialize("Team A", {"Player 1", "Player 2", "Player 3", "Player 4", "Player 5", 
                  "Player 6", "Player 7", "Player 8", "Player 9", "Player 10", "Player 11"}, 10);

    // Button positions
    Rectangle dotBallButton = {50, 500, 100, 40};
    Rectangle addRunsButton = {200, 500, 100, 40};
    Rectangle extraRunsButton = {350, 500, 100, 40};
    Rectangle wicketButton = {500, 500, 100, 40};

    // GUI Loop
    while (!WindowShouldClose()) {
        BeginDrawing();
        ClearBackground(RAYWHITE);

        // Display score details
        DrawText(game.getScoreDetails().c_str(), 20, 20, 20, DARKGRAY);

        // Draw Buttons
        DrawRectangleRec(dotBallButton, LIGHTGRAY);
        DrawText("Dot Ball", dotBallButton.x + 10, dotBallButton.y + 10, 20, BLACK);

        DrawRectangleRec(addRunsButton, LIGHTGRAY);
        DrawText("Add Runs", addRunsButton.x + 10, addRunsButton.y + 10, 20, BLACK);

        DrawRectangleRec(extraRunsButton, LIGHTGRAY);
        DrawText("Extra Runs", extraRunsButton.x + 10, extraRunsButton.y + 10, 20, BLACK);

        DrawRectangleRec(wicketButton, LIGHTGRAY);
        DrawText("Wicket", wicketButton.x + 20, wicketButton.y + 10, 20, BLACK);

        // Button Logic
        if (IsMouseButtonPressed(MOUSE_LEFT_BUTTON)) {
            Vector2 mouse = GetMousePosition();

            if (CheckCollisionPointRec(mouse, dotBallButton)) {
                game.registerDotBall();
            }
            if (CheckCollisionPointRec(mouse, addRunsButton)) {
                game.registerRuns(1); // Hardcoded for 1 run, could add input for runs
            }
            if (CheckCollisionPointRec(mouse, extraRunsButton)) {
                game.registerExtras(1); // Hardcoded for 1 extra run
            }
            if (CheckCollisionPointRec(mouse, wicketButton)) {
                game.registerWicket();
            }
        }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}

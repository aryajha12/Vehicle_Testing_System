# Vehicle_Testing_System

#include <iostream>
#include <vector>
#include <string>
#include <algorithm> // For std::find
#include <fstream>   // For database file handling
#include <random>    // For predictive testing simulation
using namespace std;

// Base Class for Vehicle
class Vehicle {
protected:
    string name;
    int maxLoad;
    vector<string> terrainCapability;
    int durability;

public:
    Vehicle(string name, int maxLoad, vector<string> terrainCapability, int durability)
        : name(name), maxLoad(maxLoad), terrainCapability(terrainCapability), durability(durability) {}

    string getName() const { return name; }
    int getMaxLoad() const { return maxLoad; }
    const vector<string>& getTerrainCapability() const { return terrainCapability; }
    int getDurability() const { return durability; }

    void improveMaxLoad(int increment) { maxLoad += increment; }
    void improveDurability(int increment) { durability += increment; }
    void addTerrainCapability(const string& terrain) { terrainCapability.push_back(terrain); }
};

// Base Class for Test Scenarios
class TestScenario {
protected:
    Vehicle* vehicle;

public:
    TestScenario(Vehicle* vehicle) : vehicle(vehicle) {}
    virtual ~TestScenario() {}

    virtual void performTest() = 0;
    virtual string getImprovementSuggestion() = 0;
    virtual string getResultForDatabase() = 0;
};

// Derived Classes for Specific Test Scenarios
class LoadTest : public TestScenario {
private:
    int load;

public:
    LoadTest(Vehicle* vehicle, int load) : TestScenario(vehicle), load(load) {}

    void performTest() override {
        bool result = load <= vehicle->getMaxLoad();
        cout << "Test: Load Test\n";
        cout << "Vehicle: " << vehicle->getName() << "\n";
        cout << "Load: " << load << "\n";
        cout << "Result: " << (result ? "Pass" : "Fail") << "\n";
        if (!result) {
            cout << "Reason: Load exceeded maximum capacity.\n";
        }
        cout << string(50, '-') << endl;
    }

    string getImprovementSuggestion() override {
        return "Increase max load capacity.";
    }

    string getResultForDatabase() override {
        return "Load Test," + to_string(load) + "," + (load <= vehicle->getMaxLoad() ? "Pass" : "Fail");
    }
};

class TerrainTest : public TestScenario {
private:
    string terrain;

public:
    TerrainTest(Vehicle* vehicle, string terrain) : TestScenario(vehicle), terrain(terrain) {}

    void performTest() override {
        bool result = find(vehicle->getTerrainCapability().begin(), vehicle->getTerrainCapability().end(), terrain) != vehicle->getTerrainCapability().end();
        cout << "Test: Terrain Test\n";
        cout << "Vehicle: " << vehicle->getName() << "\n";
        cout << "Terrain: " << terrain << "\n";
        cout << "Result: " << (result ? "Pass" : "Fail") << "\n";
        if (!result) {
            cout << "Reason: Terrain not supported.\n";
        }
        cout << string(50, '-') << endl;
    }

    string getImprovementSuggestion() override {
        return "Add support for terrain: " + terrain + ".";
    }

    string getResultForDatabase() override {
        return "Terrain Test," + terrain + "," + (find(vehicle->getTerrainCapability().begin(), vehicle->getTerrainCapability().end(), terrain) != vehicle->getTerrainCapability().end() ? "Pass" : "Fail");
    }
};

class DurabilityTest : public TestScenario {
private:
    int iterations;

public:
    DurabilityTest(Vehicle* vehicle, int iterations) : TestScenario(vehicle), iterations(iterations) {}

    void performTest() override {
        bool result = iterations <= vehicle->getDurability();
        cout << "Test: Durability Test\n";
        cout << "Vehicle: " << vehicle->getName() << "\n";
        cout << "Iterations: " << iterations << "\n";
        cout << "Result: " << (result ? "Pass" : "Fail") << "\n";
        if (!result) {
            cout << "Reason: Exceeded durability threshold.\n";
        }
        cout << string(50, '-') << endl;
    }

    string getImprovementSuggestion() override {
        return "Increase durability rating.";
    }

    string getResultForDatabase() override {
        return "Durability Test," + to_string(iterations) + "," + (iterations <= vehicle->getDurability() ? "Pass" : "Fail");
    }
};

// Class for Test Results and Suggestions
class TestResultManager {
private:
    vector<string> results;
    Vehicle* vehicle;

public:
    TestResultManager(Vehicle* vehicle) : vehicle(vehicle) {}

    void addResult(const string& result) {
        results.push_back(result);
    }

    void saveToDatabase(const string& filename) {
        ofstream file(filename, ios::app);
        for (const auto& result : results) {
            file << result << endl;
        }
        file.close();
    }

    void processImprovements(vector<TestScenario*>& tests) {
        for (auto* test : tests) {
            string suggestion = test->getImprovementSuggestion();
            if (suggestion.find("max load") != string::npos) {
                vehicle->improveMaxLoad(100); // Example improvement
            } else if (suggestion.find("durability") != string::npos) {
                vehicle->improveDurability(500); // Example improvement
            } else if (suggestion.find("terrain") != string::npos) {
                size_t pos = suggestion.find(": ");
                if (pos != string::npos) {
                    string terrain = suggestion.substr(pos + 2);
                    vehicle->addTerrainCapability(terrain);
                }
            }
        }
        cout << "\nImprovements Applied:\n";
        cout << "- Max Load: " << vehicle->getMaxLoad() << "\n";
        cout << "- Durability: " << vehicle->getDurability() << "\n";
        cout << "- Supported Terrains: ";
        for (const auto& terrain : vehicle->getTerrainCapability()) {
            cout << terrain << ", ";
        }
        cout << "\n";
    }

    void predictiveTestingSimulation() {
        random_device rd;
        mt19937 gen(rd());
        uniform_int_distribution<> loadDist(500, 1500);
        uniform_int_distribution<> durabilityDist(3000, 7000);
        vector<string> terrains = {"Off-road", "Mountain", "Sand", "Highway"};

        cout << "\nPredictive Testing Simulation:\n";
        for (int i = 0; i < 5; ++i) {
            int simulatedLoad = loadDist(gen);
            int simulatedDurability = durabilityDist(gen);
            string simulatedTerrain = terrains[gen() % terrains.size()];

            cout << "Simulation " << (i + 1) << "\n";
            cout << "- Simulated Load: " << simulatedLoad << "\n";
            cout << "- Simulated Durability: " << simulatedDurability << "\n";
            cout << "- Simulated Terrain: " << simulatedTerrain << "\n";
            cout << string(50, '-') << endl;
        }
    }
};

// Sample Execution 
int main() {
    // Create Vehicles
    Vehicle vehicle1("Truck A", 1000, {"Off-road", "Highway"}, 5000);

    // Run Tests
    TestResultManager resultManager(&vehicle1);

    vector<TestScenario*> tests = {
        new LoadTest(&vehicle1, 1200),
        new TerrainTest(&vehicle1, "Mountain"),
        new DurabilityTest(&vehicle1, 4500)
    };

    for (auto* test : tests) {
        test->performTest();
        resultManager.addResult(test->getResultForDatabase());
    }

    // Save results to database
    resultManager.saveToDatabase("test_results.txt");

    // Process Improvements
    resultManager.processImprovements(tests);

    // Run Predictive Testing Simulation
    resultManager.predictiveTestingSimulation();

    for (auto* test : tests) {
        delete test;
    }

    return 0;
}

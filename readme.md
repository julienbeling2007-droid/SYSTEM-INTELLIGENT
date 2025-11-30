# DESCRIPTION DU PROJET DE SIMULATION D'UN ECOSYSTEME INTELLIGENT 😁😁😁#
* bonjour a tous le projet suivant consistait a simuler un ecosysteme intelligent où on retrouve des entites tels que: des plantes, des animaux(carnivores, herbivores ) se mangeant entre eux formant ainsi une chaine alimentaire comme le suit carnivore -> herbivore -> plantes . MERCI POUR VOTRE ATTENTION ! *

# ce projet necessite des connaissances pointilleuses sur la notion de poo (classe, objet, encapsulation, constructeurs, destructeurs etc...) et en sdl3 pour le coté parlant du graphique de la chose #

# STRUCTURE ET DEROULEMENT DU PROJET ECOSYSTEME #
#include <cstdint>
#include <string>
#include <cmath>
/ 🏷 STRUCTS POUR LES DONNÉES SIMPLES
struct Vector2D {
    float x;
    float y;
    
    // 🏗 Constructeur avec valeurs par défaut
    Vector2D(float xValue = 0.0f, float yValue = 0.0f) : x(xValue), y(yValue) {}
    
    // 📐 Méthodes utilitaires
    float Distance(const Vector2D& other) const {
        float dx = x - other.x;
        float dy = y - other.y;
        return std::sqrt(dx * dx + dy * dy);
    }
    
    Vector2D operator+(const Vector2D& other) const {
        return Vector2D(x + other.x, y + other.y);
    }
    
    Vector2D operator*(float scalar) const {
        return Vector2D(x * scalar, y * scalar);
    }
};

struct Color {
    uint8_t r;
    uint8_t g;
    uint8_t b;
    uint8_t a;
    
    // 🏗 Constructeurs multiples
    Color() : r(255), g(255), b(255), a(255) {}  // Blanc par défaut
    
    Color(uint8_t red, uint8_t green, uint8_t blue, uint8_t alpha = 255) 
        : r(red), g(green), b(blue), a(alpha) {}
    
    // 🎨 Couleurs prédéfinies
    static Color Red() { return Color(255, 0, 0); }
    static Color Green() { return Color(0, 255, 0); }
    static Color Blue() { return Color(0, 0, 255); }
    static Color Yellow() { return Color(255, 255, 0); }
};

struct Food {
    Vector2D position;
    float energyValue;
    Color color;
    
    // 🏗 Constructeur
    Food(Vector2D pos, float energy = 25.0f) 
        : position(pos), energyValue(energy), color(Color::Green()) {}
        
        # STRUCT.H #
#include "Structs.h"
#include <SDL3/SDL.h>
#include <memory>
#include <random>
#include <vector>

namespace Ecosystem {
namespace Core {

// 🎯 ÉNUMÉRATION DES TYPES D'ENTITÉS
enum class EntityType {
    HERBIVORE,
    CARNIVORE,
    PLANT
};

class Entity {
private:
    // 🔒 DONNÉES PRIVÉES - État interne protégé
    float mEnergy;
    float mMaxEnergy;
    int mAge;
    int mMaxAge;
    bool mIsAlive;
    Vector2D mVelocity;
    EntityType mType;
    
    // 🎲 Générateur aléatoire
    mutable std::mt19937 mRandomGenerator;

public:
    // 🔓 DONNÉES PUBLIQUES - Accès direct sécurisé
    Vector2D position;
    Color color;
    float size;
    std::string name;

    // 🏗 CONSTRUCTEURS
    Entity(EntityType type, Vector2D pos, std::string entityName = "Unnamed");
    Entity(const Entity& other);  // Constructeur de copie
    
    // 🗑 DESTRUCTEUR
    ~Entity();

    // ⚙️ MÉTHODES PUBLIQUES
    void Update(float deltaTime);
    void Move(float deltaTime);
    void Eat(float energy);
    bool CanReproduce() const;
    std::unique_ptr<Entity> Reproduce();
    void ApplyForce(Vector2D force);
    
    // 📊 GETTERS - Accès contrôlé aux données privées
    float GetEnergy() const { return mEnergy; }
    float GetEnergyPercentage() const { return mEnergy / mMaxEnergy; }
    int GetAge() const { return mAge; }
    bool IsAlive() const { return mIsAlive; }
    EntityType GetType() const { return mType; }
    Vector2D GetVelocity() const { return mVelocity; }
    
    // 🎯 MÉTHODES DE COMPORTEMENT
    Vector2D SeekFood(const std::vector<Food>& foodSources) const;
    Vector2D AvoidPredators(const std::vector<Entity>& predators) const;
    Vector2D StayInBounds(float worldWidth, float worldHeight) const;
    
    // 🎨 MÉTHODE DE RENDU
    void Render(SDL_Renderer* renderer) const;

private:
    // 🔐 MÉTHODES PRIVÉES - Logique interne
    void ConsumeEnergy(float deltaTime);
    void Age(float deltaTime);
    void CheckVitality();
    Vector2D GenerateRandomDirection();
    Color CalculateColorBasedOnState() const;
};
# ENTITY .H #

#include <SDL3/SDL.h>
#include <string>
#include "../Core/Structs.h"

namespace Ecosystem {
namespace Graphics {

class Window {
private:
    // 🔒 RESSOURCES SDL
    SDL_Window* mWindow;
    SDL_Renderer* mRenderer;
    float mWidth;
    float mHeight;
    bool mIsInitialized;
    std::string mTitle;

public:
    // 🏗 CONSTRUCTEUR/DESTRUCTEUR
    Window(const std::string& title, float width, float height);
    ~Window();

    // ⚙️ INITIALISATION
    bool Initialize();
    void Shutdown();
    
    // 🎨 RENDU
    void Clear(const Core::Color& color = Core::Color(30, 30, 30));
    void Present();
    
    // 📊 GETTERS
    SDL_Renderer* GetRenderer() const { return mRenderer; }
    bool IsInitialized() const { return mIsInitialized; }
    float GetWidth() const { return mWidth; }
    float GetHeight() const { return mHeight; }
    std::string GetTitle() const { return mTitle; }
};
# WINDOW.H #

#include "../Graphics/Window.h"
#include "Ecosystem.h"
#include <chrono>

namespace Ecosystem {
namespace Core {

class GameEngine {
private:
    // 🔒 ÉTAT DU MOTEUR
    Graphics::Window mWindow;
    Ecosystem mEcosystem;
    bool mIsRunning;
    bool mIsPaused;
    float mTimeScale;
    
    // ⏱ CHRONOMÉTRE
    std::chrono::high_resolution_clock::time_point mLastUpdateTime;
    float mAccumulatedTime;

public:
    // 🏗 CONSTRUCTEUR
    GameEngine(const std::string& title, float width, float height);
    
    // ⚙️ MÉTHODES PRINCIPALES
    bool Initialize();
    void Run();
    void Shutdown();
    
    // 🎮 GESTION D'ÉVÉNEMENTS
    void HandleEvents();
    void HandleInput(SDL_Keycode key);

private:
    // 🔐 MÉTHODES INTERNES
    void Update(float deltaTime);
    void Render();
    void RenderUI();
};

} // namespace Core
} // namespace Ecosystem
# GAME ENGINE .H #

#pragma once

#include "Entity.h"
#include "Structs.h"
#include <vector>
#include <memory>
#include <random>

namespace Ecosystem {
namespace Core {

class Ecosystem {
private:
    // 🔒 ÉTAT INTERNE
    std::vector<std::unique_ptr<Entity>> mEntities;
    std::vector<Food> mFoodSources;
    float mWorldWidth;
    float mWorldHeight;
    int mMaxEntities;
    int mDayCycle;
    
    // 🎲 Générateur aléatoire
    std::mt19937 mRandomGenerator;
    
    // 📊 STATISTIQUES
    struct Statistics {
        int totalHerbivores;
        int totalCarnivores;
        int totalPlants;
        int totalFood;
        int deathsToday;
        int birthsToday;
    } mStats;

public:
    // 🏗 CONSTRUCTEUR/DESTRUCTEUR
    Ecosystem(float width, float height, int maxEntities = 500);
    ~Ecosystem();

    // ⚙️ MÉTHODES PUBLIQUES
    void Initialize(int initialHerbivores, int initialCarnivores, int initialPlants);
    void Update(float deltaTime);
    void SpawnFood(int count);
    void RemoveDeadEntities();
    void HandleReproduction();
    void HandleEating();
    
    // 📊 GETTERS
    int GetEntityCount() const { return mEntities.size(); }
    int GetFoodCount() const { return mFoodSources.size(); }
    Statistics GetStatistics() const { return mStats; }
    float GetWorldWidth() const { return mWorldWidth; }
    float GetWorldHeight() const { return mWorldHeight; }
    
    // 🎯 MÉTHODES DE GESTION
    void AddEntity(std::unique_ptr<Entity> entity);
    void AddFood(Vector2D position, float energy = 25.0f);
    
    // 🎨 RENDU
    void Render(SDL_Renderer* renderer) const;

private:
    // 🔐 MÉTHODES PRIVÉES
    void UpdateStatistics();
    void SpawnRandomEntity(EntityType type);
    Vector2D GetRandomPosition();
    void HandlePlantGrowth(float deltaTime);
};

} // namespace Core
} // namespace Ecosystem
#  ECOSYSTEME.H #




# ARBORESCENCE DU PROJET
SYSTEM INTELLIGENT/
├── include/
│   ├── Core/
│   │   ├── Structs.hpp
│   │   ├── Entity.hpp
│   │   └── Ecosystem.hpp
│   └── Graphics/
│       └── Window.hpp
├── src/
│   ├── Core/
│   │   ├── Entity.cpp
│   │   └── Ecosystem.cpp
│   ├── Graphics/
│   │   └── Window.cpp
│   └── main.cpp
├── assets/
│   └── (futures textures)
└── README.md


# MERCI POUR VOTRE ECOUTE ET BONNE APPETIT POUR LA LECTURE #

# CORDIALEMENT BELING JULIEN FILIERE AN ING1 DEBUTANT EN PROGRAMMATION C++😁😁😁😒😁😊 #

# PROJET PROPOSE PAR ING TEUGUIA SEDERIS RODOLPH #

A RETENIR :
programmer c'est de l'art


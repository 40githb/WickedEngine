# Phase 1: Engine Fork and Foundation (Months 1-2)

## Overview

Transform Wicked Engine into a soccer-dev-friendly codebase with an intuitive editor UI and decoupled ball physics working. This is the foundation everything else builds on.

**Team Assignment:**
- **You (Lead)**: Editor UI, tooling, architecture
- **Co-dev**: Physics integration, ball entity, force application
- **Both**: Daily syncs, shared API design

---

## Week 1-2: Setup & Architecture

### You: Editor UI & Tooling Foundation

**Goal**: Integrate Dear ImGui and create the editor framework.

**Tasks:**

- [ ] **Fork Wicked Engine** (if not done)
  - GitHub fork: https://github.com/turanszkij/WickedEngine
  - Clone locally: `git clone https://github.com/YOUR-USERNAME/WickedEngine.git wicked-soccer`

- [ ] **Integrate Dear ImGui**
  - Add `cmake/FindImGui.cmake` (module finder)
  - Update root `CMakeLists.txt` to link ImGui
  - Test: Build succeeds, no linker errors
  - Commit: `feature/imgui-integration`

- [ ] **Create SoccerEditor base class**
  - File: `src/tools/editor/SoccerEditor.h/cpp`
  - Responsibility: Main ImGui window manager
  - Interface:
    ```cpp
    class SoccerEditor {
    public:
        void Init(wiGraphics::GraphicsDevice* device);
        void Update(float deltaTime);
        void Render();
        
    private:
        void DrawMenuBar();
        void DrawDockingArea();
        
        ImGuiID dockspaceID;
        std::vector<std::unique_ptr<EditorPanel>> panels;
    };
    ```
  - Commit: `feature/soccer-editor-base`

- [ ] **Create EditorPanel interface**
  - File: `src/tools/editor/EditorPanel.h`
  - Responsibility: Base class for all editor windows (PhysicsDebugger, AnimationInspector, etc.)
  - Interface:
    ```cpp
    class EditorPanel {
    public:
        virtual ~EditorPanel() = default;
        virtual void Update(float deltaTime) = 0;
        virtual void Draw() = 0;  // ImGui draw calls
        virtual const char* GetName() const = 0;
    };
    ```
  - Commit: `feature/editor-panel-base`

- [ ] **Documentation**
  - Create `docs/ARCHITECTURE.md` (overview of folder structure)
  - Create `docs/PHASE_1_CHECKLIST.md` (this file)
  - Commit: `docs/initial-architecture`

**Dependencies**: None (foundational)

**Deliverable**: `wicked-soccer` repo with ImGui integrated, SoccerEditor ready to extend.

**Done when**: Build succeeds, ImGui window renders in the editor.

---

### Co-dev: Physics Integration & Ball Entity

**Goal**: Set up PhysX, create the Ball entity, basic physics.

**Tasks:**

- [ ] **PhysX SDK setup**
  - Download NVIDIA PhysX SDK 5.x
  - Create `cmake/FindPhysX.cmake` (module finder)
  - Update root `CMakeLists.txt` to link PhysX libraries
  - Test: Build succeeds, PhysX headers include without errors
  - Commit: `feature/physx-integration`

- [ ] **PhysXIntegration wrapper**
  - File: `src/soccer/physics/PhysXIntegration.h/cpp`
  - Responsibility: Manage PhysX scene, materials, substeps
  - Interface:
    ```cpp
    class PhysXIntegration {
    public:
        void Init();
        void Update(float deltaTime);
        void Shutdown();
        
        PxScene* GetScene() { return scene; }
        PxMaterial* CreateMaterial(float staticFriction, float dynamicFriction, float restitution);
        
    private:
        PxFoundation* foundation;
        PxPhysics* physics;
        PxScene* scene;
        PxCooking* cooking;
    };
    ```
  - Commit: `feature/physx-wrapper`

- [ ] **Ball entity and basic physics**
  - File: `src/soccer/physics/Ball.h/cpp`
  - Responsibility: Represents the ball in the game world
  - Properties:
    ```cpp
    class Ball {
    public:
        void Init(wiScene::Scene& scene, const glm::vec3& startPosition);
        void Update(float deltaTime);
        
        glm::vec3 GetPosition() const;
        glm::vec3 GetVelocity() const;
        glm::vec3 GetAngularVelocity() const;
        float GetMass() const { return 0.43f; }  // Official soccer ball mass in kg
        
        void ApplyForce(const glm::vec3& force);
        void ApplyTorque(const glm::vec3& torque);
        
        // Callback when player foot touches ball
        struct TouchEvent {
            glm::vec3 contactPoint;
            glm::vec3 contactNormal;
            float contactForce;
        };
        std::function<void(const TouchEvent&)> OnFootContact;
        
    private:
        PxRigidDynamic* rigidBody;
        glm::vec3 velocity;
        glm::vec3 angularVelocity;
    };
    ```
  - Commit: `feature/ball-entity`

- [ ] **Basic collision detection**
  - Implement PhysX collision callbacks in Ball class
  - Fire `OnFootContact` callback when player touches ball
  - Commit: `feature/ball-collisions`

- [ ] **Documentation**
  - Create `docs/PHYSICS_DESIGN.md`
    - Overview of ball physics system
    - PhysX configuration details
    - Pitch materials (grass friction, restitution)
  - Commit: `docs/physics-design`

**Dependencies**: ImGui (for future debugging tools)

**Deliverable**: Ball entity with PhysX physics, ready for force application.

**Done when**: Build succeeds, ball spawns in the scene and reacts to gravity.

---

### Both: Sync on APIs & Dependencies

**Goal**: Align physics/editor APIs so they can work together seamlessly.

**Tasks:**

- [ ] **Design Ball API**
  - Agree on `Ball` class interface (above)
  - Discuss `TouchEvent` structure
  - Commit: `feature/ball-api-design`

- [ ] **Design Physics Debugger interface**
  - What does the editor need to display ball state?
  - Velocity vectors? Spin rate? Contact points?
  - Example:
    ```cpp
    // In Ball.h
    struct DebugInfo {
        glm::vec3 position;
        glm::vec3 velocity;
        glm::vec3 angularVelocity;
        float spinRateMagnitude;
        std::vector<glm::vec3> recentContactPoints;
    };
    DebugInfo GetDebugInfo() const;
    ```
  - Commit: `feature/ball-debug-interface`

- [ ] **Daily standup** (10 min, async or voice)
  - "What did I finish?"
  - "What am I starting?"
  - "Anything blocking me?"
  - Document in `docs/STANDUP.md` or Slack

**Deliverable**: Shared understanding of how systems connect.

---

## Week 3-4: Core Integrations

### You: Physics Debugger & Editor Panels

**Goal**: Create editor UI for visualizing and debugging ball physics.

**Tasks:**

- [ ] **PhysicsDebugger editor panel**
  - File: `src/tools/editor/PhysicsDebugger.h/cpp`
  - Renders as an ImGui window in the editor
  - Displays:
    - Ball position (XYZ sliders)
    - Ball velocity (magnitude + vector visualization)
    - Angular velocity (spin rate, axis)
    - Recent contact points
    - PhysX scene statistics
  - Example ImGui code:
    ```cpp
    void PhysicsDebugger::Draw() {
        if (!ImGui::Begin("Physics Debugger", nullptr)) {
            ImGui::End();
            return;
        }
        
        Ball::DebugInfo info = game->GetBall().GetDebugInfo();
        
        ImGui::Text("Position: %.2f, %.2f, %.2f", 
            info.position.x, info.position.y, info.position.z);
        ImGui::Text("Velocity: %.2f m/s", glm::length(info.velocity));
        ImGui::Text("Spin Rate: %.0f RPM", info.spinRateMagnitude);
        
        // Visualization
        if (ImGui::CollapsingHeader("Visualization")) {
            ImGui::Checkbox("Show velocity vector", &drawVelocityVector);
            ImGui::Checkbox("Show contact points", &drawContactPoints);
        }
        
        ImGui::End();
    }
    ```
  - Commit: `feature/physics-debugger`

- [ ] **Test scene loader**
  - File: `src/soccer/core/SceneLoader.h/cpp`
  - Loads `assets/scenes/test_pitch.scene`
  - Spawns ball at center, enables gravity
  - Commit: `feature/test-scene-loader`

- [ ] **Main editor window layout**
  - Use ImGui docking to arrange:
    - Top: Menu bar (File, Edit, View, Help)
    - Left: Scene hierarchy (object tree)
    - Center: 3D viewport (WE renderer)
    - Right: Properties/Inspector
    - Bottom: Physics Debugger, console
  - Commit: `feature/editor-layout`

- [ ] **Testing**
  - Load test_pitch.scene
  - Ball should be visible and falling
  - PhysicsDebugger shows correct values
  - Commit: `test/physics-debug-validation`

**Dependencies**: Co-dev's Ball entity, PhysXIntegration

**Deliverable**: Editor with PhysicsDebugger panel, test scene loads correctly.

**Done when**: Open editor, see ball falling, inspect values in PhysicsDebugger.

---

### Co-dev: Magnus Effect & Ball Forces

**Goal**: Implement Magnus effect and allow applying forces to the ball.

**Tasks:**

- [ ] **BallPhysics class**
  - File: `src/soccer/physics/BallPhysics.h/cpp`
  - Responsibility: Calculate forces to apply to ball each frame
  - Interface:
    ```cpp
    class BallPhysics {
    public:
        struct Config {
            float airDensity = 1.225f;  // kg/m^3
            float ballRadius = 0.11f;   // meters
            float ballMass = 0.43f;
            float dragCoefficient = 0.7f;
        };
        
        BallPhysics(const Config& config);
        
        // Apply each frame during physics update
        void Update(Ball& ball, float deltaTime);
        
    private:
        // Magnus effect: spinning ball curves
        glm::vec3 CalculateMagnusForce(const glm::vec3& velocity, const glm::vec3& angularVelocity);
        
        // Air drag: opposes motion
        glm::vec3 CalculateDragForce(const glm::vec3& velocity);
        
        Config config;
    };
    ```
  - Commit: `feature/ball-physics-system`

- [ ] **Magnus effect implementation**
  - Research: Magnus force formula (already in your design doc)
  - Formula: `F_magnus = (0.5 * rho * A * Cl * v^2) * (omega x v_unit)`
  - Where:
    - `rho` = air density
    - `A` = cross-section area
    - `Cl` = lift coefficient (varies with spin rate)
    - `omega` = angular velocity
    - `v_unit` = velocity direction
  - Implementation notes:
    - Cl peaks at ~2500 RPM, reduces at higher spins (realistic)
    - Only applies if spin rate > ~1500 RPM
  - Commit: `feature/magnus-effect`

- [ ] **Drag force implementation**
  - Opposes velocity, proportional to v^2
  - Formula: `F_drag = -0.5 * rho * A * Cd * v^2 * v_unit`
  - Commit: `feature/drag-force`

- [ ] **Pitch material properties**
  - File: `src/soccer/physics/PitchMaterial.h`
  - Different areas have different friction/restitution:
    ```cpp
    enum class PitchType {
        Grass,      // High friction, low restitution
        Mud,        // Even higher friction, very low bounce
        BareBare,   // Lower friction, medium restitution
    };
    ```
  - Create PhysX materials for each
  - Commit: `feature/pitch-materials`

- [ ] **Testing**
  - Create test: Ball with spin should curve
  - Create test: Heavy drag should slow ball down
  - Commit: `test/magnus-and-drag-tests`

**Dependencies**: Ball entity (from Week 1-2)

**Deliverable**: Ball physics system with Magnus effect and drag.

**Done when**: Spinning ball visibly curves in the test scene.

---

### Both: Integration & Iteration

**Tasks:**

- [ ] **Daily builds**
  - Both push to `main` only via reviewed PRs
  - CI should auto-build on push (set up GitHub Actions)
  - Commit: `.github/workflows/build.yml`

- [ ] **Test the integration**
  - Open editor
  - Load test_pitch.scene
  - Ball falls and curves correctly
  - PhysicsDebugger shows accurate values

- [ ] **Code review**
  - Review each other's PRs
  - Merge when approved

**Deliverable**: Clean builds, physics working visually.

---

## Week 5-8: Features & Iteration

### You: More Editor Tools

**Goal**: Build out editor to make tweaking easier.

**Tasks:**

- [ ] **Ball properties editor**
  - Editor panel to tweak:
    - Mass, radius, drag coefficient
    - Air density (simulate altitude/weather)
    - Restitution/friction
  - Save/load presets
  - Commit: `feature/ball-properties-editor`

- [ ] **Console & logging**
  - File: `src/tools/editor/Console.h/cpp`
  - Displays build + physics debug messages
  - Commit: `feature/editor-console`

- [ ] **Scene saver**
  - Save current ball state to file
  - Load previously saved states
  - Useful for testing edge cases
  - Commit: `feature/scene-saver`

- [ ] **Performance profiler**
  - ImGui window showing FPS, physics tick time, render time
  - Commit: `feature/profiler-panel`

**Dependencies**: PhysicsDebugger, ball properties system

**Deliverable**: Comprehensive editor tooling for tweaking physics.

---

### Co-dev: Pitch Tuning & Surface Interaction

**Goal**: Make ball behavior realistic across different pitch conditions.

**Tasks:**

- [ ] **Surface friction system**
  - Ball loses more momentum on wet grass than dry
  - Implement friction coefficient per surface type
  - Commit: `feature/surface-friction`

- [ ] **Ball rolling simulation**
  - When ball is on ground with spin, it rolls
  - Gradually converts spin into linear motion
  - Commit: `feature/rolling-dynamics`

- [ ] **Bounce behavior**
  - Restitution varies by pitch type
  - Grass absorbs energy (low bounce)
  - Bare ground bounces more
  - Commit: `feature/bounce-tuning`

- [ ] **Ball collisions with environment**
  - Ball bounces off goalposts, stands, etc.
  - Implement PhysX collision groups
  - Commit: `feature/environmental-collisions`

- [ ] **Tuning pass**
  - Tweak all coefficients until behavior *feels right*
  - Compare to FIFA/PES reference videos
  - Commit: `feature/physics-tuning`

**Dependencies**: BallPhysics system, pitch materials

**Deliverable**: Ball behavior matches reference games, tweakable via editor.

---

## Week 9-10: Polish & Testing

### Both: QA & Documentation

**Tasks:**

- [ ] **Build validation**
  - [ ] Build on Windows (MSVC)
  - [ ] Build on Linux (GCC)
  - [ ] No warnings
  - Commit: `ci/cross-platform-build`

- [ ] **Automated tests**
  - [ ] PhysicsTests.cpp: Ball falls with correct acceleration
  - [ ] PhysicsTests.cpp: Magnus force direction is correct
  - [ ] PhysicsTests.cpp: Drag slows the ball
  - Commit: `test/physics-unit-tests`

- [ ] **Integration test**
  - [ ] Load test_pitch.scene
  - [ ] Verify ball spawns, falls, curves
  - Commit: `test/scene-loading-test`

- [ ] **Documentation finalization**
  - [ ] ARCHITECTURE.md complete
  - [ ] PHYSICS_DESIGN.md complete
  - [ ] CONTRIBUTING.md reviewed
  - Commit: `docs/phase-1-docs`

- [ ] **Performance baseline**
  - [ ] Record FPS with ball physics at 120Hz
  - [ ] Identify bottlenecks
  - Commit: `perf/baseline-metrics`

**Deliverable**: Clean, documented, tested codebase ready for Phase 2.

---

## Week 11-12: Wrap-Up & Phase 2 Prep

### Both: Retrospective & Planning

**Tasks:**

- [ ] **Phase 1 retrospective**
  - What went well?
  - What was hard?
  - What would you do differently?
  - Document in `docs/RETROSPECTIVE_PHASE_1.md`

- [ ] **Phase 2 preparation**
  - Create `docs/PHASE_2_CHECKLIST.md` (animation system)
  - List required animations
  - Motion capture data sources
  - Motion Matching algorithm research

- [ ] **Code cleanup**
  - Remove TODOs that aren't real
  - Dead code deletion
  - Final refactoring pass
  - Commit: `refactor/phase-1-cleanup`

- [ ] **Final PR review and merge**
  - Merge all feature branches to `main`
  - Tag release: `git tag -a phase1-complete -m "Phase 1 complete"`
  - Commit: `release/phase-1`

**Deliverable**: Clean main branch, ready for Phase 2.

---

## Success Criteria for Phase 1

By the end of Week 12, you should have:

✅ Wicked Engine fork with Dear ImGui integrated
✅ SoccerEditor framework with docking layout
✅ Ball entity with PhysX physics
✅ Magnus effect and drag forces implemented
✅ Editor tools for physics debugging and tweaking
✅ Test scene that demonstrates ball physics
✅ Clean, documented codebase with CI/CD
✅ Both developers comfortable with the codebase

---

## Common Issues & Solutions

**Build fails with PhysX linking errors:**
- Ensure PhysX SDK is installed in standard location
- Update `cmake/FindPhysX.cmake` with correct paths
- On Windows: Use MSVC 2019+
- On Linux: Use GCC 10+

**PhysicsDebugger window crashes:**
- Check that Ball entity is always valid
- Add null checks: `if (!ball) return;`
- Ensure ball is updated every frame before debugger draws

**Ball physics doesn't feel right:**
- Tweak drag coefficient (0.5-0.9)
- Adjust Magnus Cl coefficient
- Compare to reference videos (FIFA 19, PES 2021)
- Use editor sliders to iterate quickly

**Merge conflicts on Ball.h:**
- Communication is key—talk before coding
- Split ownership: Physics owns velocity/forces, Animation owns animation state
- Use feature flags while both are working

---

## Timeline Summary

| Week | You | Co-dev | Deliverable |
|------|-----|--------|-------------|
| 1-2 | ImGui, SoccerEditor | PhysX, Ball entity | Editor UI + ball physics base |
| 3-4 | PhysicsDebugger | Magnus effect | Physics debugging + curved ball |
| 5-8 | Editor tools | Ball tuning | Tweakable physics, ref-game-accurate |
| 9-10 | Testing & docs | Testing & docs | Clean, tested codebase |
| 11-12 | Retrospective | Retrospective | Phase 1 complete, Phase 2 ready |

---

## Questions or Blockers?

Anything unclear? Reach out before you start a task—asking early saves days of rework.

Good luck! 🚀

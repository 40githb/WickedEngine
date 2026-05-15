# Contributing to Wicked Soccer

Welcome! This document explains how to work on the wicked-soccer fork as a team.

## Quick Start

```bash
# Clone the repo
git clone https://github.com/YOUR-USERNAME/wicked-soccer.git
cd wicked-soccer

# Create a feature branch
git checkout main
git pull origin main
git checkout -b feature/your-feature-name

# Code, commit, push
git add .
git commit -m "Description of change"
git push origin feature/your-feature-name

# Open a Pull Request on GitHub
# Get review from team
# Merge when approved
```

---

## Git Workflow (Tight Collaboration)

Because you're collaborating tightly on shared systems, follow this **to the letter**:

### Branch Naming

```
feature/physics-magnus-effect       # Feature (subsystem/feature)
bugfix/animation-blend-crash        # Bug fix
docs/physics-design                 # Documentation
refactor/imgui-integration          # Refactoring
```

### Before Starting Work

**Always sync with latest main:**

```bash
git checkout main
git pull origin main
```

### Committing

**Keep commits atomic and descriptive:**

```bash
# GOOD
git commit -m "Implement Magnus effect calculation in BallPhysics

- Add spin vector to ball entity
- Calculate aerodynamic force using air density
- Integrate with PhysX scene query"

# BAD
git commit -m "physics stuff"
git commit -m "fix"
```

### Pushing & Pull Requests

**After your feature is working:**

```bash
git push origin feature/your-feature-name
```

Go to GitHub and create a Pull Request:
- **Title**: One-line summary
- **Description**: 
  - What changed and why
  - Any breaking changes
  - Testing instructions (if applicable)
  - Related issues (if any)

**Example PR:**
```
Title: Add Magnus effect to ball physics

Description:
Implements spinning ball aerodynamics per design doc Section 2.1.

Changes:
- BallPhysics::CalculateMagnusForce() now considers spin vector
- PhysX material properties updated for different pitch surfaces
- Ball curves realistically during volleys and crosses

Testing:
- Build and run WickedEngine_Editor
- Load test_pitch.scene
- Shoot a curved ball; it should bend in flight

Closes #42
```

### Code Review

**If you're the reviewer:**
1. Read the code diff on GitHub
2. Test locally if it touches physics/animation
3. Comment on specific lines: `Does this handle edge case X?`
4. Approve or request changes
5. Merge when satisfied

**If you're the author:**
1. Address feedback in new commits
2. Push the same branch (GitHub auto-updates PR)
3. Request re-review when done

### Merging

**Only merge on GitHub (via PR).** Never `git merge` locally and push.

Click "Merge Pull Request" on GitHub. Choose one:
- **Squash and merge** (single commit, cleaner history)
- **Create a merge commit** (preserves individual commits)

For wicked-soccer: **Use "Squash and merge"** to keep history clean.

### After Merge

```bash
# Delete local branch
git branch -d feature/your-feature-name

# Pull latest main on both machines
git checkout main
git pull origin main
```

---

## Code Style

### C++ Guidelines

- **Naming**: `camelCase` for variables/functions, `PascalCase` for classes
- **Header files**: `.h` extension
- **Implementation**: `.cpp` extension
- **Indentation**: 4 spaces (no tabs)
- **Line length**: Max 100 characters

**Example:**

```cpp
// Ball.h
class Ball {
public:
    glm::vec3 GetVelocity() const { return velocity; }
    void ApplyForce(const glm::vec3& force);
    
private:
    glm::vec3 velocity;
    glm::vec3 angularVelocity;
    float mass;
};

// Ball.cpp
void Ball::ApplyForce(const glm::vec3& force) {
    velocity += (force / mass) * deltaTime;
}
```

### Comments

- **Why**, not what. Code shows *what*; comments explain *why*.

```cpp
// GOOD
// Magnus effect reduces at low spin rates (below ~1500 RPM)
// to simulate realistic aerodynamics
if (spinRate < kMinSpinForMagnus) {
    magnusForce *= spinRate / kMinSpinForMagnus;
}

// BAD
// Check spin rate
if (spinRate < kMinSpinForMagnus) {
    // Reduce force
    magnusForce *= spinRate / kMinSpinForMagnus;
}
```

### File Headers

Start every `.cpp` and `.h` file with:

```cpp
/*
    SoccerGame - Module Name
    
    Responsibility: One or two sentences describing what this file does
    
    Collaborators: BallPhysics, AnimationSystem (if you depend on others)
*/

#pragma once  // For .h files

// Includes
#include "core/Ball.h"
#include <glm/glm.hpp>

// Your code...
```

---

## Collaboration Rules for Tight Integration

### 1. **Daily Syncs on Shared Systems**

If you're both working on physics + animation integration:

```bash
# Every morning (or before starting)
git pull origin main

# This ensures you're not duplicating work
```

### 2. **Define Interfaces First**

Before coding, discuss the boundary between subsystems:

**Example**: Physics + Animation integration

```cpp
// In Ball.h (Physics subsystem)
class Ball {
public:
    struct TouchEvent {
        glm::vec3 contactPoint;
        glm::vec3 contactNormal;
        float contactForce;
    };
    
    // Callback fired when player foot touches ball
    std::function<void(const TouchEvent&)> OnFootContact;
};

// In SoccerAnimationComponent.cpp (Animation subsystem)
void SoccerAnimationComponent::OnBallTouched(const Ball::TouchEvent& event) {
    // Play kick animation based on force/angle
}
```

**Both of you agree on `TouchEvent` structure before coding separately.**

### 3. **Use Feature Flags for Incomplete Work**

If you're working on something incomplete that affects shared code:

```cpp
#define FEATURE_MOTION_MATCHING 1

#if FEATURE_MOTION_MATCHING
    // New motion matching code
    animator->MatchMotion(targetPose);
#else
    // Old fallback code
    animator->SetAnimationState(kRunning);
#endif
```

This lets you work on features without breaking the build for your co-dev.

### 4. **Conflict Resolution**

If you both edit the same file:

```bash
# You pushed first
git pull origin main
git checkout feature/your-branch
git rebase origin/main

# Git marks conflicts. Open the file and:
# 1. Keep the best parts of both versions
# 2. Test that it still builds
# 3. Commit the resolved merge

git add .
git rebase --continue
git push -f origin feature/your-branch
```

**Talk to each other** if conflicts happen—merge is a signal you're stepping on toes.

---

## Testing Before You Push

### Build Locally

```bash
cd build
cmake ..
make -j4
```

### Run Tests

```bash
# After build succeeds
./bin/tests  # Run unit tests (when you add them)
```

### Manual Testing

```bash
./WickedEngine_Editor
# Open test_pitch.scene
# Test your feature manually
```

### Don't Push If

- ❌ Build fails
- ❌ Your feature is incomplete (use feature flags if you must)
- ❌ You didn't test it
- ❌ You skipped a code review

---

## Documentation

### When to Document

- **New public API**: Add comments to `.h` file
- **Complex algorithm**: Add `ARCHITECTURE.md` section
- **New subsystem**: Create a design doc in `docs/`
- **Breaking changes**: Update `README.md`

### Example: Documenting a Function

```cpp
/*
    ApplyMagnusForce
    
    Applies aerodynamic force to the ball based on its spin.
    
    Parameters:
    - ball: The ball entity to modify
    - windVector: Environmental wind (optional)
    
    Formula: F = (0.5 * rho * A * Cl * v^2) where:
      - rho: air density (~1.225 kg/m^3 at sea level)
      - A: ball cross-section area
      - Cl: lift coefficient (depends on spin)
      - v: velocity magnitude
    
    Returns: The calculated force vector (for logging/debugging)
    
    Notes:
    - Called every physics tick (120Hz)
    - Disabled if spin rate < 1500 RPM (realistic threshold)
    - Thread-safe; can be called from physics thread
*/
glm::vec3 BallPhysics::ApplyMagnusForce(Ball& ball, const glm::vec3& windVector);
```

---

## Phase 1 Workflow (First 2 Months)

### Week 1-2: Setup & Planning
- **You**: Integrate Dear ImGui, set up editor UI framework
- **Co-dev**: Set up PhysX, create Ball entity with basic physics
- **Both**: Daily syncs on API boundaries

### Week 3-4: Core Integrations
- **You**: PhysicsDebugger window in ImGui, ball visualization
- **Co-dev**: Ball-player collision detection, force application
- **Both**: Test that ball reacts to player input

### Week 5-8: Features & Iteration
- **You**: Add more editor panels (PlayerStatsEditor, etc.)
- **Co-dev**: Magnus effect, pitch friction, surface properties
- **Both**: Continuously integrate and test

### End of Phase 1
- Build compiles cleanly
- Editor UI is intuitive
- Ball physics feels "correct"
- Ready for animation system (Phase 2)

---

## Questions?

If something is unclear, ask **before** you code. Collaboration saves time only when everyone knows the plan.

Good luck! 🚀

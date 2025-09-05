# Humanoid Locomotion via Primitive Composition: A Data-Driven Approach

This repository accompanies the Master's Thesis:

**“Humanoid Locomotion via Primitive Composition: a Data-Driven Approach.”**

The project explores robust bipedal locomotion for humanoid robots without relying on perfectly known analytical models. Instead, it composes a set of data-driven control primitives (policies) obtained via model-based reinforcement learning (MBRL). These primitives are treated as a "crowd" of candidate behaviors; a probabilistic crowdsourcing algorithm selects and blends them online inside the MuJoCo physics simulation environment.

### Demo
Humanoid locomotion rollout:

![Humanoid locomotion demo]([./demonstration.mp4](https://github.com/user-attachments/assets/254cd82b-05a9-4176-921f-144ad972edcf))

The approach targets adaptability in uncertain or dynamic environments by:
- Leveraging diverse pretrained or learned policies (primitive library)
- Maintaining uncertainty-aware state representations
- Dynamically scoring and selecting policies based on task-specific costs
- Enabling scalable extension to new tasks and control modalities

---
### Attribution (TD-MPC2)
This work builds upon the TD-MPC2 framework by **Nicklas Hansen, Hao Su, and Xiaolong Wang**.

- Original [TD-MPC2 repository](https://github.com/nicklashansen/tdmpc2).
- Policies used as primitives in this thesis were trained or adapted in a [companion repository](https://github.com/msc-thesis-workgroup/tdmpc2-for-Unitree-H1) I maintain. That repository extends TD-MPC2 for Unitree H1 humanoid locomotion and contains the modified training pipeline and scripts used to produce the policies composed here. This repository focuses on the composition layer (crowdsourcing, selection logic, task integration); training is intentionally isolated for clarity and maintainability.

The TD-MPC2 components (world model, latent planning, scaling, buffers) form the backbone for generating the control primitives that are then composed by the crowdsourcing layer developed in this thesis. Proper credit to the original authors should be maintained in any derivative or extended work.

---
## 1. Core Ideas
1. **Primitive Composition** – Rather than a monolithic controller, multiple policy services compete/cooperate.
2. **Crowdsourcing Mechanism** – A selection/composition loop evaluates expected performance under uncertainty.
3. **Model-Based RL Foundation** – Primitives originate from TD-MPC components.
4. **Simulation Fidelity** – MuJoCo enables realistic dynamics for rapid iteration and evaluation.
5. **Modularity** – Clear boundaries between tasks, services, world models, and orchestration.

---
## 2. Repository Structure
```
data_driven_legged_locomotion/
	__main__.py              # Entry: builds environment + runs crowdsourcing loop
	TDMPC_crowdsourcing.py   # High-level orchestration integrating TDMPC-like primitives
	common/                  # Crowdsourcing logic: services set, state space, probabilistic filtering
	agents/                  # Policy service wrappers + Hybrid / TDMPC agent code
		tdmpc/                 # World model, buffer, layers, math utils, scaling, parser
		config*/               # YAML configs + pretrained checkpoints (*.pt)
	tasks/                   # Task-specific domains (pendulum, humanoid walking)
	utils/                   # Logging, CSV helpers, plotting notebook
demonstration.mp4          # Example rollout
requirements.txt           # Python dependencies
```

Key folders:
- `common/`: Core abstractions – `Crowdsourcing`, `ServiceSet`, `StatePF`, `StateSpace`.
- `agents/`: Service implementations, hybrid controller logic, TDMPC components.
- `agents/tdmpc/common/`: World model components, scaling, buffers, math utilities.
- `tasks/h1_walk/`: Humanoid locomotion task & cost shaping utilities.
- `utils/`: Logging utilities (`logging.py`), CSV parsing.

---
## 3. Method Overview
1. **Primitive Generation**: Model-based RL (TD-MPC) produces candidate controllers with a learned world model and planning routine.
2. **Service Wrapping**: Each controller is exposed as a *service* supplying action proposals and metadata.
3. **Crowdsourcing Loop**:
	 - Maintain probabilistic belief/state filtering (`StatePF`, `StateSpace`).
	 - Score services via cost or predicted improvement.
	 - Compose or select an action (e.g., weighted or arg-min selection) for the environment step.
4. **Adaptation**: Service weights evolve with observed trajectories and performance.
5. **Evaluation**: Executed in MuJoCo for physically plausible feedback.

---
## 4. Installation
Tested with Python 3.11 (3.10+ likely compatible).

```bash
git clone https://github.com/my-rice/Humanoid-Locomotion-via-primitive-composition.git
cd Humanoid-Locomotion-via-primitive-composition
python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

MuJoCo dependencies: ensure system packages for OpenGL / EGL / GLFW are present. For headless servers:
```bash
export MUJOCO_GL=egl
```

---
## 5. Quick Start
Run the main crowdsourcing experiment:
```bash
python -m data_driven_legged_locomotion
```

Currently, environment composition and task selection are configured manually inside `data_driven_legged_locomotion/__main__.py`.

---
## 6. Logging & Analysis
- Logging setup: `utils/logging.py`
- CSV utilities: `utils/read_csv.py`

Extend logging by adding handlers or structured metrics where services or crowdsourcing decisions are executed.

---
## 7. Extending the Framework
Add a new primitive service:
1. Implement a service wrapper in `agents/` (extend existing hybrid or TDMPC base patterns).
2. Register it in `TDMPC_crowdsourcing.py` or the orchestration logic in `__main__.py`.

Add a new task:
1. Create a folder `tasks/<your_task>/`.
2. Provide MuJoCo XML (if needed) + cost/reward shaping script.
3. Integrate selection logic or cost tests similar to `h1_walk/` and `pendulum/`.

Modify selection strategy:
- Adjust or extend scoring in `common/Crowdsourcing.py`.

Swap world model:
- Implement or modify components in `agents/tdmpc/common/world_model.py` and update configuration YAML.

---
## 8. Experimental Metrics (Typical)
- Stability (fall / termination rate)
- Goal achievement (task-specific success)
- Diversity utilization (entropy over selected services)

---
## 9. License
Distributed under the terms specified in `LICENSE`.

---
## 10. Contact
For questions or collaboration, open an Issue or reach the thesis author (add email / profile).

---
## 11. Summary
This project delivers a modular framework that composes model-based RL primitives through a probabilistic crowdsourcing mechanism, enabling adaptive humanoid locomotion in MuJoCo while remaining extensible for new tasks, controllers, and research directions.

---
### Short TL;DR
Composable data-driven primitives + uncertainty-aware selection → robust humanoid locomotion in simulation.


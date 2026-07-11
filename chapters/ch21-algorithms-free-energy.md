# Chapter 21: Algorithms — Free Energy & Enhanced Sampling

## Core Idea
GROMACS supports multiple free energy methods: thermodynamic integration (TI), free energy perturbation (FEP), Bennett's acceptance ratio (BAR), and replica exchange. Energy minimization uses steepest descent (initial) and conjugate gradient or L-BFGS (refinement).

## Energy Minimization

| Method | Algorithm | Parallel | Use |
|---|---|---|---|
| Steepest descent | Gradient-based, robust | Yes | Initial relaxation |
| Conjugate gradient | Polak-Ribiere CG | Yes | Refinement near minimum |
| L-BFGS | Quasi-Newton | No (yet) | Fastest convergence |

- `emtol = 10.0 kJ/mol/nm` — stop when max force < tolerance
- `emstep = 0.01 nm` — initial step size
- CG + steep hybrid: `nstcgsteep = 1000` (do steep every 1000 CG steps)

## Free Energy Calculations

### Thermodynamic Integration (TI)
- ΔG = ∫₀¹ ⟨∂H/∂λ⟩_λ dλ
- Lambda: coupling parameter (0=initial, 1=final state)
- Soft-core potentials prevent endpoint singularities
- `sc-alpha = 0.5`, `sc-power = 1`, `sc-sigma = 0.3`

### Free Energy Perturbation (FEP)
- ΔG = −k_B T ln⟨exp(−ΔH/k_B T)⟩
- Forward and reverse estimates bracket the true value
- BAR combines both for optimal estimate

### Bennett's Acceptance Ratio (BAR)
- Optimal combination of forward and reverse FEP
- `gmx bar -f dhdl1.xvg dhdl2.xvg ... -g ener1.edr ener2.edr ... -temp 300`

### Alchemical Free Energy Workflow
1. Set `free-energy = yes` in .mdp
2. Define `couple-moltype` (which molecule decouples)
3. Set `couple-lambda0 = vdw-q`, `couple-lambda1 = none`
4. Define lambda schedule: `init-lambda`, `delta-lambda`
5. Run 20+ lambda windows in parallel with `-multidir`
6. Analyze with `gmx bar`

## Enhanced Sampling

### Replica Exchange
- Multiple replicas at different temperatures/hamiltonians
- Exchanges attempted every `nst-replex` steps
- Temperature: enhances sampling of high-barrier regions
- Hamiltonian: exchanges lambda values for alchemical transitions

### Essential Dynamics
- PCA-based sampling along collective eigenvectors
- Amplifies motions along essential subspace

### Expanded Ensemble
- Single simulation samples multiple states (lambda, temperature)
- Weight hopping or Wang-Landau for state weights
- Checkpoint issue: known problem in 2026.2

## Normal Mode Analysis
- Requires double precision compilation
- `integrator = nm` in .mdp
- Computes eigenvalues/eigenvectors of Hessian matrix
- Use for: vibrational analysis, entropy estimation, flexibility assessment

## Key Takeaways
1. Steepest descent → CG/L-BFGS: two-stage EM strategy
2. Soft-core potentials are essential for alchemical free energy
3. BAR/MBAR gives optimal free energy estimates
4. 20+ lambda windows typically needed for converged ΔG
5. Replica exchange helps overcome high energy barriers

## Connects To
- **Ch 25**: Soft-core potential details
- **Ch 28**: Pull code free energy methods
- **Ch 29**: AWH adaptive biasing
- **Ch 39**: Advanced free energy MDP parameters

## 中文术语对照 (Chinese Terminology)

**自由能计算理论** (来自中文手册 §3.12):

| 中文 | English | 说明 |
|------|---------|------|
| 自由能计算 | Free energy calculation | 几种方法: 慢增长, TI, BAR |
| 慢增长方法 | Slow growth method | λ从0缓慢变到1, 积分功=ΔG (理想情况) |
| 热力学积分 | Thermodynamic Integration (TI) | 在离散λ点计算⟨∂H/∂λ⟩, 积分得ΔG |
| 耦合参数 | Coupling parameter λ | λ=0为状态A, λ=1为状态B |
| 热力学循环 | Thermodynamic cycle | ΔG₁−ΔG₂ = ΔG₃−ΔG₄, 间接计算有意义的量 |
| 配分函数 | Partition function (Q, Δ) | A=−kBT·lnQ, G=−kBT·lnΔ |
| 哈密顿量 | Hamiltonian H(p,q;λ) | λ依赖的函数形式, 对各力场贡献不同 |
| Helmholtz自由能 | Helmholtz free energy (A) | NVT系综, A=−kBT·lnQ |
| Gibbs自由能 | Gibbs free energy (G) | NPT系综, G=−kBT·lnΔ, G=A+pV |
| 副本交换动力学 | Replica exchange dynamics (REMD) | 多个副本在不同温度/哈密顿量下交换构型 |
| 主成分动力学抽样 | Essential Dynamics / PCA dynamics | 沿PCA集体模式采样 |
| 扩展系综动力学 | Expanded ensemble dynamics | 单次模拟中采样多个λ态 |
| 简正模式分析 | Normal mode analysis | 对角化质量加权的Hessian矩阵 |
| 能量最小化 | Energy minimization | 最速下降(steep)→共轭梯度(CG)→L-BFGS |

**慢增长方法的局限性**: 正向和反向增长结果相等"并不能确保结果的正确性"——只是必要而非充分条件。实际中需要无限缓慢的变化, 不实用。

**λ向量的灵活性** (中文手册 §6.1): 从GROMACS 4.6开始, λ是向量, 库仑/vdW/键合/约束分量可独立控制。例如先关闭库仑项(前4个窗口), 然后用软核势关闭vdW(后6个窗口):
```
coul-lambdas      = 0.0  0.2  0.5  1.0  1.0  1.0  1.0  1.0  1.0  1.0
vdw-lambdas       = 0.0  0.0  0.0  0.0  0.4  0.5  0.6  0.7  0.8  1.0
```

Sources: GROMACS 5.0.2 中文手册 (李继存译) §3.12, §6.1, CC-BY compatible.

## Practical Workflows (Lemkul Tutorials)

### vdW Decoupling: Methane in Water
The simplest alchemical FE calculation — turn off only LJ interactions (charges already zero). Based on Shirts et al. (2003) reference data.

**Directory structure per lambda window:**
```
Lambda_0/{EM,NVT,NPT,Production_MD}
Lambda_1/{EM,NVT,NPT,Production_MD}
...
Lambda_20/{EM,NVT,NPT,Production_MD}
```

**Key .mdp settings:**
```ini
free-energy          = yes
couple-moltype       = Methane       ; must match [ moleculetype ] name
couple-lambda0       = vdw-q
couple-lambda1       = none
couple-intramol      = no
sc-alpha             = 0.5
sc-power             = 1
sc-sigma             = 0.3
nstdhdl              = 10
; 21 windows, Δλ=0.05
vdw-lambdas          = 0.00 0.05 0.10 0.15 0.20 0.25 0.30 0.35 0.40 0.45 0.50 0.55 0.60 0.65 0.70 0.75 0.80 0.85 0.90 0.95 1.00
```

**Analysis:**
```bash
gmx bar -f Lambda_*/Production_MD/md*_dhdl.xvg -o bar.xvg
# Reference: ΔG_vdW ≈ 9.0 kJ/mol (Shirts et al.)
```

### Two-Stage Hydration Free Energy: Ethanol
Computing complete ΔG_hydr requires sequential decoupling:

1. **Stage 1 — Coulomb discharge** (20 windows, Δλ=0.05):
   - `couple-lambda0 = q` (start charged)
   - `couple-lambda1 = none` (end with zero charge)
   - vdW stays ON throughout (prevents atomic overlap → infinite forces)

2. **Stage 2 — vdW decoupling** (20 windows, Δλ=0.05):
   - `couple-lambda0 = vdw` (start with full vdW, charges already zero)
   - `couple-lambda1 = none`

3. **Result**: ΔG_hydr = ΔG_coul + ΔG_vdW
   - Validate against experiment (ethanol ΔG_hydr ≈ −20 kJ/mol with OPLS)

**Critical rule — always discharge BEFORE decoupling vdW:**
If vdW is removed first, oppositely charged atoms approach at zero distance → singularities in the Coulomb potential → simulation blow-up.

**Efficiency tip — restart from previous window:**
```bash
# Each lambda i starts from lambda i-1's equilibrated output
coorfile=$prevlambda/Production_MD/md$prevlambda.gro
```
This uses the final configuration of λ_i as starting point for λ_{i+1}, accelerating equilibration.

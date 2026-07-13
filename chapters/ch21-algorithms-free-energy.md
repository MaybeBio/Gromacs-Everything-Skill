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

| 中文 | English | Notes |
|------|---------|-------|
| 自由能计算 | Free energy calculation | 慢增长、热力学积分(TI)、Bennett接受率(BAR) |
| 热力学循环 | Thermodynamic cycle | ΔG₁−ΔG₂=ΔG₃−ΔG₄，间接计算无法直接测量的ΔG |
| 耦合参数 | Coupling parameter λ | λ=0→状态A, λ=1→状态B |
| λ向量 | Lambda vector | 自4.6起，库仑/vdW/键合/约束分量可独立控制 |
| 慢增长方法 | Slow growth method | λ缓慢从0→1，路径必须可逆(理想情况无限慢) |
| 热力学积分 | Thermodynamic Integration (TI) | ΔG=∫₀¹⟨∂H/∂λ⟩dλ，在离散λ点精确求值 |
| Bennett接受率 | BAR (Bennett's Acceptance Ratio) | 最优组合正向和反向FEP结果 |
| MBAR | Multistate BAR | 同时计算多个λ态的ΔG，优于BAR |
| 亥姆霍兹自由能 | Helmholtz free energy A | A=−k_B T·lnQ，NVT系综 |
| 吉布斯自由能 | Gibbs free energy G | G=−k_B T·lnΔ，G=A+pV，NPT系综 |
| 配分函数 | Partition function (Q, Δ) | 相空间积分，MD模拟无法直接计算 |
| NVT→NPT校正 | NVT→NPT correction | G(p)−G(p₀)=A(V)−A(V₀)−∫[V(p')−V]dp' |
| 动能质量贡献 | Kinetic mass contribution | −(3/2)k_B T·ln(m^B/m^A)，仅无约束时适用 |
| 副本交换分子动力学 | Replica Exchange MD (REMD) | 多副本在不同温度/哈密顿量下定期交换构型 |
| 温度副本交换 | Temperature REMD | 交换概率 P=min(1, exp[(1/k_B T₁−1/k_B T₂)(U₁−U₂)]) |
| 哈密顿副本交换 | Hamiltonian REMD | 不同λ值的交换，由自由能路径定义 |
| 等温等压REMD | NPT REMD | 交换概率包含体积项：(P₁/k_B T₁−P₂/k_B T₂)(V₁−V₂) |
| Gibbs采样REMD | Gibbs sampling REMD | 允许交换不相邻副本，测试所有可能配对 |
| 本性动力学采样 | Essential dynamics sampling | 沿PCA集体模式约束/增强采样 |
| 自由能导数的系综平均 | Ensemble average of ∂H/∂λ | ⟨∂H/∂λ⟩_{NVT;λ} = dA/dλ, ⟨∂H/∂λ⟩_{NpT;λ} = dG/dλ |

**关键概念**:
1. **慢增长方法的局限**: 正向和反向增长结果相等的说法是必要非充分条件——"正向和反向增长结果的相等并不能保证结果的正确性"。实际上需要无限缓慢的变化，不实用
2. **TI方法**: 设delta_lambda=0，每次在固定λ值进行平衡+采样，通过∂H/∂λ的涨落确定dG/dλ的误差估计，然后数值积分得总ΔG
3. **REMD温度选择**: 对于简谐势c≈1，蛋白质/水时c≈2。接受概率P≈exp(−ε²·c·N_df/2)，期望P≈0.135时ε≈2/√(c·N_df)。所有键受约束时N_df≈2N_atoms，故ε≈1/√N_atoms
4. **REMD交换策略**: 奇数步交换奇数副本对(0-1, 2-3)，偶数步交换偶数副本对(1-2)，避免交换依赖问题

**重要公式**:
- 热力学积分: ΔG^{B→A}=∫₀¹ ⟨∂H/∂λ⟩_{NpT;λ} dλ
- 自由能与配分函数: A=−k_B T·lnQ, G=−k_B T·lnΔ, G=A+pV
- 导数的系综平均: dA/dλ=⟨∂H/∂λ⟩_{NVT;λ}, dG/dλ=⟨∂H/∂λ⟩_{NpT;λ}
- REMD接受概率: P=min(1, exp[(1/k_B T₁−1/k_B T₂)(U₁−U₂)])
- NPT REMD: P=min(1, exp[(U₁−U₂)(1/k_B T₁−1/k_B T₂)+(P₁/k_B T₁−P₂/k_B T₂)(V₁−V₂)])

Sources: GROMACS 2019.6 中文译版, §5.4.11-5.4.14

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

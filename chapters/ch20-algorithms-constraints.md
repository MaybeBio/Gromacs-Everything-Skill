# Chapter 20: Algorithms — Constraints & Coupling

## Core Idea
GROMACS provides multiple thermostat and barostat options for different simulation phases, plus three constraint algorithms (LINCS, SHAKE, SETTLE) for bond length constraints. The key principle: use V-rescale + Parrinello-Rahman for production, Berendsen for fast initial relaxation.

## Temperature Coupling

### Berendsen Thermostat
- **How**: Velocities scaled by λ = [1 + Δt/τ_T (T₀/T − 1)]^½
- **When**: Fast pre-equilibration
- **Why not production**: Does NOT produce correct canonical ensemble
- **tau_t**: 0.1 ps typical

### V-rescale (Velocity Rescaling)
- **How**: Berendsen-like scaling with stochastic correction term
- **When**: Equilibration and production (NVT or NPT)
- **Why**: Produces CORRECT canonical ensemble; more robust than Nose-Hoover
- **tau_t**: 0.1 ps (equilibration), 1.0-2.0 ps (production)

### Nose-Hoover
- **How**: Extended system with thermal reservoir, oscillatory relaxation
- **When**: Production with velocity Verlet integrator
- **Why**: Correct ensemble; can oscillate with poor settings
- **nh-chain-length**: 10 (default), longer chain = better but more expensive

### Andersen
- **How**: Stochastic velocity reassignment at random times
- **When**: Simple thermostat, rarely used in production

### Stochastic Dynamics (SD)
- **How**: Langevin equation: m·dv/dt = F − m·γ·v + R(t)
- **When**: Implicit solvent, coarse-grained simulations
- **tau_t**: 2 ps gives friction < water internal friction

## Pressure Coupling

### Berendsen Barostat
- **How**: Coordinates and box vectors scaled toward target pressure
- **When**: Fast initial box relaxation
- **Why not production**: Does NOT produce correct NPT ensemble
- **tau_p**: 2 ps typical
- **compressibility**: 4.5e-5 bar⁻¹ (water)

### Parrinello-Rahman
- **How**: Extended system with box vectors as dynamical variables
- **When**: Production NPT
- **Why**: Correct NPT ensemble; can oscillate — ensure stable box first
- **tau_p**: 5 ps typical
- **pcoupltype**: isotropic (solution), semiisotropic (membrane), anisotropic, surface-tension

### MTTK (Martyna-Tuckerman-Tobias-Klein)
- **How**: Fully correct integration of NPT equations
- **When**: When fully correct NPT ensemble is critical

## Constraint Algorithms

### LINCS (LINear Constraint Solver)
- **Default**, fastest, well-parallelized
- **lincs-order**: 4 (expansion order for coupling matrix)
- **lincs-iter**: 1 (correction iterations)
- **lincs-warnangle**: 30° (warning threshold for bond rotation)

### SHAKE
- Iterative Lagrange multiplier method
- Slower than LINCS, less parallel
- Uses relative tolerance: `shake-tol = 0.0001`

### SETTLE
- Analytical solution for rigid 3-atom water molecules
- Fastest possible, auto-applied for rigid water models
- No user parameters needed

## Simulated Annealing
- Time-dependent reference temperature via annealing groups
- Multiple annealing groups with different schedules
- Use `annealing = single` or `periodic`

## Key Takeaways
1. V-rescale for thermostat, Parrinello-Rahman for barostat in production
2. Berendsen methods are for equilibration only — wrong ensemble
3. LINCS is the default and recommended constraint algorithm
4. SETTLE is automatic for rigid water — no configuration needed
5. SD integrator with tau_t=2 ps is a good Langevin thermostat

## Connects To
- **Ch 9**: MDP coupling parameters
- **Ch 19**: MD integrator details

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 温度耦合 | Temperature coupling / Thermostat | 从NVE→NVT系综，修改运动方程 |
| 弱耦合 | Weak coupling | Berendsen方法特征，一阶指数衰减，不产生正确系综 |
| 速度重缩放 | Velocity rescaling (V-rescale) | 附加随机项的Berendsen恒温器，产生正确正则系综 |
| Nosé-Hoover恒温器 | Nosé-Hoover thermostat | 扩展系综方法，引入热浴变量ξ，振荡弛豫 |
| 热浴变量/摩擦参数 | Heat bath variable ξ | Nosé-Hoover中的额外自由度，p_ξ为其动量 |
| Nosé-Hoover链 | Nosé-Hoover chain | 链长默认为10，改进遍历性但仍非完全遍历 |
| 质量参数 | Mass parameter Q | Q = τ_T²T₀/(4π²)，决定Nosé-Hoover耦合强度 |
| Andersen恒温器 | Andersen thermostat | 周期性从Maxwell-Boltzmann分布重选速度，减慢动力学 |
| 恒压器/压浴 | Barostat / Pressure bath | Berendsen, Parrinello-Rahman, MTTK |
| 等温压缩系数 | Isothermal compressibility β | 水: 4.6×10⁻⁵ bar⁻¹ (7.6×10⁻⁴ MD单位) |
| Parrinello-Rahman耦合 | P-R barostat | 扩展系统，产生真正NPT系综，时间常数应为弱耦合的4-5倍 |
| 半各向同性缩放 | Semiisotropic scaling | x/y方向各向同性 + z方向独立，用于膜系统 |
| 完全各向异性变形 | Fully anisotropic deformation | 使用约束时需更缓慢缩放或减小时间步长 |
| 表面张力耦合 | Surface tension coupling | γ(t) = (L_z/n)[P_zz − (P_xx+P_yy)/2] |
| MTTK压力控制 | Martyna-Tuckerman-Tobias-Klein | 速度Verlet专用的NPT方法，5.1起仅用于无约束系统 |
| 约束算法 | Constraint algorithms | LINCS(默认)、SHAKE、SETTLE |
| SHAKE | SHAKE | 迭代Lagrange乘子，相对容差收敛，并行性差 |
| SETTLE | SETTLE | 刚性水分子分析解，超过80%模拟体系涉及 |
| RATTLE | RATTLE | 速度Verlet第二步约束，去除平行于键向量的速度分量 |
| LINCS | LINCS | 非迭代两步法，矩阵幂展开求逆，比SHAKE更快更稳定 |
| P-LINCS | P-LINCS | LINCS的并行版本，处理跨单元边界的键约束 |
| 约束耦合矩阵 | Constraint coupling matrix | B_n M⁻¹ B_n^T，K×K矩阵，对角含1/m₁+1/m₂ |
| 展开阶数 | Expansion order (lincs-order) | MD四阶，大时间步长BD需八阶 |
| 约束三角形 | Constraint triangles | 醇类基团OH键角约束特征值~0.7，收敛较慢 |
| 拉格朗日乘子 | Lagrange multiplier λ_k | 求解约束力，G_i = −Σ_k λ_k·∂σ_k/∂r_i |
| 模拟退火 | Simulated annealing (SA) | 多组不同退火策略，支持single和periodic |
| 随机动力学 | Stochastic dynamics (SD) / Langevin | m·dv/dt = −mγv + F + 噪声力 |
| 布朗动力学 | Brownian dynamics (BD) | 过阻尼极限，忽略惯性：dr/dt = F/γ + 噪声 |
| 最陡下降 | Steepest descent (SD) | 沿负梯度(力方向)移动，稳健但收敛慢 |
| 共轭梯度法 | Conjugate gradient (CG) | 远离极小点表现差，但近极小点高效 |
| L-BFGS | Limited-memory BFGS | 准牛顿方法，滑动窗校正，收敛最快但未并行化 |
| 简正模式分析 | Normal mode analysis (NMA) | 对角化质量加权Hessian矩阵，需双精度 |
| Hessian矩阵 | Hessian matrix | H_ij = ∂²V/∂x_i∂x_j，3N×3N矩阵 |

**关键概念**:
1. **恒温器选择原则**: Berendsen(快速弛豫，不产生正确系综)→仅用于初始平衡; V-rescale(速度重缩放+随机项)→产生正确NVT系综，推荐生产使用; Nosé-Hoover(扩展系统，振荡弛豫)→τ_T应为弱耦合的4-5倍; Andersen→减慢动力学，通常不用于研究输运性质
2. **恒压器选择原则**: Berendsen(快速盒子弛豫)→仅用于初始; P-R(扩展系统，真正NPT)→需接近平衡密度再用，τ_p取4-5倍; 膜系统必须semiisotropic; MTTK仅与速度Verlet+无约束使用
3. **LINCS核心**: 矩阵A_n的特征值~0.4(仅键约束)或~0.7(含角约束三角形)。每增加一阶偏差降为原来的0.4。算法不可能崩溃，即使无法重置约束也会产生尽可能满足约束的构型
4. **简正分析前提**: 需完全能量最小化(容差~0.001 kJ/mol)，需双精度。使用最陡下降+共轭梯度或L-BFGS进行最小化，禁用约束。

**重要公式**:
- Berendsen缩放: λ = [1 + n_TC·Δt/τ_T·(T₀/T−1)]^½ (限制范围0.8≤λ≤1.25)
- Nosé-Hoover运动: d²r_i/dt² = F_i/m_i − (p_ξ/Q)·dr_i/dt
- SD更新: α = 1−e^(−γΔt), Δv = −α·v' + √(k_B T(1−α²)/m)·噪声
- 最陡下降: r_{n+1}=r_n + F_n·h_n/max(|F_n|), V↓→h×1.2, V↑→h×0.2
- Hessian对角化: R^T·M^(−½)·H·M^(−½)·R = diag(λ_1,...,λ_3N), λ_i=(2πω_i)²

Sources: GROMACS 2019.6 中文译版, §5.4.5-5.4.10

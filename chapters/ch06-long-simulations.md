# Chapter 6: Managing Long Simulations

## Core Idea
GROMACS supports robust checkpoint-based restart for long simulations. The .cpt file contains full-precision coordinates, velocities, and simulation state, enabling exact continuation. Understanding reproducibility and the difference between binary and analytical equivalence is important for production simulations.

## Frameworks Introduced
- **Checkpoint-restart pattern**: Use `-cpi state.cpt` to continue; `-noappend` to avoid overwriting output; `-cpo state_new.cpt` to name output checkpoint
- **Extension pattern**: Use `gmx convert-tpr -extend <time>` to extend completed simulation; then `gmx mdrun -s next.tpr -cpi state.cpt`

## Key Concepts

### Appending vs New Output
- By default, mdrun appends to existing output files (trajectory, energy, log)
- Use `-noappend` to start fresh output files (deletes previous output)
- Appending preserves the full simulation history in a single set of files

### Checkpoint Files (.cpt)
- Binary files containing full-precision coordinates, velocities, forces, and simulation state
- Named `state.cpt` by default; rename with `-cpo`
- Can be queried with `gmx check -f state.cpt`
- Required for exact continuation

### Extending .tpr Files
- When a simulation completes, use `gmx convert-tpr -s old.tpr -extend <time> -o next.tpr`
- Then continue: `gmx mdrun -s next.tpr -cpi state.cpt`
- This preserves the original simulation parameters while extending the runtime

### Changing MDP Options for Restart
- Create new .tpr with modified .mdp: `gmx grompp -f new.mdp -c old.gro -p topol.top -t state.cpt -o new.tpr`
- The `-t state.cpt` flag copies full-precision coordinates and velocities from checkpoint
- This allows changing parameters (e.g., switching from NVT to NPT, changing thermostat) mid-simulation

### Restarts Without Checkpoint Files
- If .cpt is lost, use last .gro frame: `gmx grompp -f new.mdp -c lastframe.gro -p topol.top -o new.tpr`
- Velocities are generated from Maxwell-Boltzmann distribution at `gen-temp`
- This is NOT an exact continuation — velocities are random

### Are Continuations Exact?
- **Exact binary reproducibility**: Requires same .tpr, same .cpt, same executable, same hardware, same number of MPI ranks, same OpenMP threads, same `-reprod` flag
- **Analytical equivalence**: Leap-frog and velocity Verlet trajectories are analytically (not binary) identical when started from corresponding points
- **Long-term divergence**: Even with exact restarts, trajectories diverge due to floating-point non-associativity in parallel runs

### Reproducibility
- Use `-reprod` flag to force deterministic execution (binary identical output on same hardware)
- Performance impact: significant reduction in parallel efficiency
- Generally NOT needed — statistical equivalence is sufficient for analysis

## Code Examples
```bash
# Standard restart from checkpoint
gmx mdrun -s topol.tpr -cpi state.cpt -noappend

# Extend completed simulation by 100 ns
gmx convert-tpr -s topol.tpr -extend 100000 -o next.tpr
gmx mdrun -s next.tpr -cpi state.cpt

# Change parameters and continue
gmx grompp -f production.mdp -c npt.gro -t state.cpt -p topol.top -o prod.tpr
gmx mdrun -s prod.tpr -cpi state.cpt

# Query checkpoint contents
gmx check -f state.cpt

# Reproducible run (same binary output on same hardware)
gmx mdrun -s topol.tpr -reprod
```

## Key Takeaways
1. Always save checkpoint files — they're your insurance against crashes
2. Use `-noappend` only when you want to start fresh output files
3. `gmx convert-tpr -extend` is the proper way to extend completed simulations
4. For exact restarts after changing parameters, use `grompp -t state.cpt`
5. Binary reproducibility requires identical hardware and parallel decomposition — use `-reprod` for debugging only
6. Statistical equivalence is sufficient for production analysis

## Connects To
- **Ch 10**: mdrun features (rerun, multi-simulation)
- **Ch 15**: Validation and reproducibility
- **Ch 27**: .cpt file format details

## 中文术语对照 (Chinese Terminology)

| 中文 | English | Notes |
|------|---------|-------|
| 长时间模拟 | long simulations | 需要检查点重启 |
| 检查点文件 | checkpoint file (.cpt) | 全精度坐标和速度 |
| 追加输出 | appending output | 默认行为 |
| 校验码 | checksum | 验证输出文件完整性 |
| 备份文件 | backup files | rsync 定期复制 |
| 延长 tpr | extending tpr | gmx convert-tpr -extend |
| 续跑 | continuation | 从检查点继续 |
| 精确续跑 | exact continuation | 二进制相同的结果 |
| 重现性 | reproducibility | 相同硬件+相同输入 |
| 混沌性 | chaotic nature of MD | 轨迹迅速发散 |
| 动态负载均衡 | dynamic load balancing | 影响可重现性 |
| 终止信号 | termination signals | SIGTERM, SIGUSR1, Ctrl-C |
| 文件截断 | file truncation | 不连续输出时 |

**关键概念**: GROMACS 支持通过检查点文件实现精确的模拟续跑。检查点文件保存全精度的位置、速度和算法状态信息，重启时可以做到与不中断的连续运行完全等价。默认情况下重启会追加到已有输出文件，使用 -noappend 可强制写入新文件。检查点文件维护每个输出文件的校验码，如果输出文件在 mdrun 写入后被修改或删除，追加会失效。模拟的精确重现需要相同可执行文件+相同硬件+相同共享库+相同输入文件+相同命令行参数。受浮点运算的不可结合性和动态负载均衡的影响，即使使用检查点续跑，并行模拟的长期轨迹也会发散。

Sources: GROMACS 2019.6 中文译版 (§3.3)

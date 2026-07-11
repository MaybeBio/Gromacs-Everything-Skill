# Test case
> ⚠️ 仅供参考, 再次声明本skill只用于加速RAG收敛，提供思路（宽度）但需要参考官方权威来源进行验证并修改

用户      ❯ 

```python 
/Gromacs-Everything-Skill 我手头上有1个CTCF蛋白质+zn配位金属离子+DNA序列，如何做模拟，好像gromacs中用不了ZAFF?
```

Skill     ❯
```python
 ---                                                                                                           
  CTCF + Zn²⁺ + DNA 在 GROMACS 中的完整模拟方案                                                                   
                                                                                                                  
  背景信息                                                                                                        
                                                                                                                  
  CTCF (CCCTC-binding factor, UniProt P49711) 是一个含有 11 个 C₂H₂ 锌指 的重要转录调节因子，约 727               
  个氨基酸。每个锌指通过 2 个 Cys 和 2 个 His 配位 1 个 Zn²⁺，共 11 个 Zn²⁺。                                     
                                                                                                                  
  ---                                                                                                             
  为什么 ZAFF 不能在 GROMACS 中直接用                                                                           
                                                                                                                  
  ZAFF (Zinc Amber Force Field) 是 AMBER 力场的参数扩展集，它定义的是 Zn 配位键的 平衡键长和力常数。它被写进 AMBER
   的 leaprc 和 frcmod 文件中，只能被 AMBER 的 tleap 读取。GROMACS 的 pdb2gmx 不认识 ZAFF 格式。                  
                                      
  但 ZAFF 的参数可以通过 间接路径 进入 GROMACS（方案 3）。                                                        
                                                                                                                
  ---                                                                                                             
  方案对比总览                                                                                                  
                                                                                                                  
  ┌──────┬───────────────────────────────────┬────────┬────────┬────────────────┬───────────────────┐           
  │ 方案 │               名称                │ 准确性 │ 工作量 │ GROMACS 兼容性 │     适合场景      │             
  ├──────┼───────────────────────────────────┼────────┼────────┼────────────────┼───────────────────┤             
  │ 1    │ 纯非键模型                        │ 低     │ 低     │ 原生           │ 快速测试          │             
  ├──────┼───────────────────────────────────┼────────┼────────┼────────────────┼───────────────────┤             
  │ 2    │ CHARMM36 + 手动成键               │ 中高   │ 中     │ 原生           │ 日常模拟，推荐    │             
  ├──────┼───────────────────────────────────┼────────┼────────┼────────────────┼───────────────────┤             
  │ 3    │ MCPB.py → ParmEd 转换             │ 高     │ 高     │ 需转换         │ 发表级精度        │             
  ├──────┼───────────────────────────────────┼────────┼────────┼────────────────┼───────────────────┤             
  │ 4    │ CHARMM36 + specbond.dat 自动识别  │ 中高   │ 低-中  │ 原生           │ CTCF 11 ZFs，推荐 │           
  ├──────┼───────────────────────────────────┼────────┼────────┼────────────────┼───────────────────┤             
  │ 5    │ 虚拟原子 (VSite) 方法             │ 高     │ 高     │ 原生           │ 追求精确配位几何  │
  ├──────┼───────────────────────────────────┼────────┼────────┼────────────────┼───────────────────┤             
  │ 6    │ 双力场混搭 (Amber蛋白+CHARMM核酸) │ 高     │ 极高   │ 有风险         │ 不推荐新手        │           
  └──────┴───────────────────────────────────┴────────┴────────┴────────────────┴───────────────────┘             
                                                                                                                
  ---                                                                                                             
  方案 1：纯非键模型（不推荐）                                                                                  
                                                                                                                  
  将 11 个 Zn²⁺ 当作自由离子加入溶剂，靠非键相互作用维持配位。                                                  
                                                                                                                  
  流程：                                                                                                          
  gmx pdb2gmx -f ctcf_dna.pdb -o complex.gro -ff charmm36 -water tip3p -ignh -ter                                 
  gmx editconf -f complex.gro -o newbox.gro -bt dodecahedron -d 1.0                                               
  gmx solvate -cp newbox.gro -cs spc216.gro -p topol.top -o solv.gro                                              
  gmx grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr                                                     
  gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral                               
                                                                                                                  
  Zn²⁺ 以独立离子存在，CHARMM36 力场自带 Zn 离子参数（ZN，电荷 +2）。                                             
                                                                                                                  
  问题： Zn²⁺ 不受方向性约束，C₂H₂ 配位几何无法维持，模拟中配位体会很快散开。对 CTCF 多达 11                      
  个锌指，锌指结构会迅速塌缩。不要用。                                                                            
                                                                                                                  
  ---                                                                                                           
  方案 2：CHARMM36 + 手动成键（推荐，CTCFe 首选）                                                               
                                                                                                                  
  核心思路：pdb2gmx 生成基础拓扑 → 手动将 Zn 并入蛋白质链的 [atoms] → 在 [bonds]
   中添加 Zn-配位原子间的谐振势。                                                                                 
                                      
  完整分步流程                                                                                                    
                                      
  Step 1：PDB 预处理                                                                                              
                                                                                                                
  CTCF 的 PDB（来自 AlphaFold3 预测或其他来源）需要做 3 件事：                                                    
                                                                                                                
  1a. DNA 原子名修正（CHARMM36 命名不匹配）                                                                       
  CHARMM36 的 DNA 残基命名与标准 PDB 不同：                                                                     
                                                                                                                  
  ┌────────────┬─────────────────┐                                                                                
  │ PDB 标准名 │ CHARMM36 rtp 名 │                                                                                
  ├────────────┼─────────────────┤                                                                                
  │ OP1        │ O1P             │    
  ├────────────┼─────────────────┤                                                                                
  │ OP2        │ O2P             │                                                                                
  ├────────────┼─────────────────┤                                                                                
  │ C7 (DT)    │ C5M             │                                                                                
  ├────────────┼─────────────────┤                                                                                
  │ O3* / O3'  │ O3'             │                                                                              
  ├────────────┼─────────────────┤                                                                                
  │ H2'1       │ H2'             │    
  ├────────────┼─────────────────┤                                                                                
  │ H2'2       │ H2''            │                                                                              
  ├────────────┼─────────────────┤                                                                                
  │ H5'1       │ H5'             │                                                                              
  ├────────────┼─────────────────┤                                                                                
  │ H5'2       │ H5''            │                                                                              
  └────────────┴─────────────────┘                                                                                
                                                                                                                
  创建 na.arn 文件放到力场目录中，定义原子名映射：                                                                
  [ ARN ]                                                                                                       
  ; pdb_name  rtp_name                                                                                            
  OP1         O1P                                                                                                 
  OP2         O2P                                                                                                 
  C7          C5M                                                                                                 
                                                                                                                  
  如果某些原子（如 OP3）在 rtp 中不存在，直接删除：                                                               
  grep -v " OP3" ctcf_dna.pdb > ctcf_clean.pdb                                                                    
                                                                                                                  
  1b. Zn 合并到单链                                                                                               
  CTCF 有 11 个 Zn²⁺，它们在 PDB 中可能被标记为不同残基。将所有 Zn 合并到一条链（Chain B），残基编号              
  1-11，去掉中间的 TER：                                                                                          
  ATOM      1  ZN  ZN  B   1        ...  ← Zn1                                                                    
  ATOM      2  ZN  ZN  B   2        ...  ← Zn2                                                                    
  ...                                                                                                             
  ATOM     11  ZN  ZN  B  11        ...  ← Zn11                                                                   
                                                                                                                  
  1c. 配位残基改名                                                                                                
  C₂H₂ 锌指中，配位的 Cys 需要去质子化为 CYM（带 -1 电荷），配位的 His 改为 HSD（H on ND1, NE2                    
  用于配位，不带额外电荷）。                                                                                      
                                                                                                                  
  确定 11 个锌指各自的 2xCys + 2xHis 在序列中的位置。以 CTCF 经典锌指排列为例（需根据你的实际 PDB 确认）：        
                                                                                                                
  ZF1:  CysXX  CysXX  HisXX  HisXX                                                                                
  ZF2:  CysXX  CysXX  HisXX  HisXX                                                                                
  ...                                                                                                             
  ZF11: CysXX  CysXX  HisXX  HisXX                                                                                
                                                                                                                  
  修改 PDB 中这些残基的名称（CYS → CYM, HIS → HSD）。                                                             
                                                                                                                  
  1d. 末端选择                                                                                                    
  运行 pdb2gmx 时需要交互选择末端类型：                                                                         
  - 蛋白 N-term: NH3+                                                                                             
  - 蛋白 C-term: COO-                                                                                             
  - DNA 5': 5PHO                                                                                                  
  - DNA 3': 3TER                                                                                                  
                                                                                                                  
  Step 2：生成基础拓扑                                                                                            
                                                                                                                  
  gmx pdb2gmx -f ctcf_clean.pdb -o complex.gro -p topol.top \                                                     
    -ff charmm36 -water tip3p -ignh -ter                                                                          
                                                                                                                  
  此时 11 个 Zn 在独立的 Ion_chain_B 中，与蛋白质没有成键。                                                       
                                                                                                                  
  Step 3：将 Zn 原子加入蛋白质拓扑                                                                                
                                                                                                                
  打开 topol_Protein_chain_A.itp，在 [ atoms ] 末尾添加 11 个 Zn 原子。假设蛋白质原有 N 个原子：                  
                                                                                                                
  ; 原子序号从 N+1 开始                                                                                           
  N+1    ZN    306   ZN    ZN1    1   2.0                                                                         
  N+2    ZN    307   ZN    ZN2    1   2.0                                                                         
  ...                                                                                                             
  N+11   ZN    316   ZN    ZN11   1   2.0                                                                         
                                                                                                                  
  Step 4：确定配位原子序号                                                                                        
                                                                                                                  
  11 个锌指 × 4 个配位原子 = 44 条配位键。需要找到每个配位原子在 [ atoms ] 中的序号：                             
                                                                                                                
  grep "CYM" topol_Protein_chain_A.itp | grep "SG"   # 22 个 Cys SG                                               
  grep "HSD" topol_Protein_chain_A.itp | grep "NE2"  # 22 个 His NE2                                              
                                                                                                                  
  Step 5：在 [ bonds ] 中添加配位键                                                                               
                                                                                                                  
  在 [ bonds ] 区域的末尾（或开头），添加 44 条配位键。使用 bond type 6（谐振势）：                               
                                      
  [ bonds ]                                                                                                       
  ; 使用 bond type 6: funct=6, 参数为 b0(nm) 和 kb(kJ/mol/nm^2)                                                   
  ; Zn_idx  Ligand_idx  funct  b0       kb                                                                        
  N+1       SG1_idx     6      0.235   50000  ; ZF1 Zn - Cys SG                                                   
  N+1       SG2_idx     6      0.235   50000  ; ZF1 Zn - Cys SG                                                   
  N+1       NE2_1_idx   6      0.210   50000  ; ZF1 Zn - His NE2                                                  
  N+1       NE2_2_idx   6      0.210   50000  ; ZF1 Zn - His NE2                                                  
  N+2       ...         ...    ...     ...    ; ZF2                                                               
  ...       ...         ...    ...     ...                                                                        
                                                                                                                  
  参数来源说明：                                                                                                  
  - b0 (平衡键长)：经验值 Zn-SG ≈ 0.235 nm，Zn-NE2 ≈ 0.210 nm。更精确的值可以从初始 PDB 结构中测量实际距离取平均：
  # 在 PDB 中量 Zn 到配位原子的距离                                                                               
  grep "ZN" ctcf.pdb | awk '{print $6}'  # 残基号                                                               
  - 或用 VMD/PyMOL 测量后取平均。                                                                                 
  - kb (力常数)：经验值 50000 kJ/mol/nm² 约等于 ~120 kcal/mol/Å²，在锌指模拟中广泛使用。                          
  - funct 1 vs 6 的区别：                                                                                         
    - type 1: 简谐键 V = 0.5 * kb * (r - b0)²，需要给出 b0, kb（但还需要角度参数）                                
    - type 6: 约束势 + 简谐键 V = 0.5 * kb * (r - b0)²，只需要 b0, kb，不需要提供角度参数。推荐使用 type 6。      
                                                                                                                  
  ▎ bond type 1 vs 6 详细解释：两者都简谐势 V = 0.5 * kb * (r - b0)²，但 type 1 是标准键（参与 1-3 排除）         
  ▎ ，type 6 是"额外键"或第三方键（不参与排除列表）。对于 Zn 跨残基配位，type 1 需要手动处理[排除]，              
  ▎ type 6 自动处理，因此更安全简便。                                                                             
                                                                                                                  
  Step 6：修改主拓扑 topol.top                                                                                    
                                                                                                                  
  - #include "topol_Ion_chain_B.itp"                                                                              
  ...                                                                                                             
  [ molecules ]                                                                                                   
  - Ion_chain_B   1    ; 删除这一行                                                                               
    Protein_chain_A  1  ; Zn 已并入蛋白                                                                           
    DNA_chain_K      1                                                                                            
    DNA_chain_L      1                                                                                            
                                                                                                                  
  同时需要把 topol_Ion_chain_B.itp 中原来引用的 ZN 的 [ atomtypes ] 参数保留。                                    
                                                                                                                  
  Step 7：修正坐标文件                                                                                            
                                                                                                                
  complex.gro 需要重新排列原子顺序：把 11 个 Zn 的坐标从文件末尾移到蛋白质原子之后。                              
                                                                                                                
  # 用脚本处理 gro 文件                                                                                           
  # 顺序: Protein_A 原子(含新增Zn) → DNA_K → DNA_L                                                                
                                                                                                                  
  Step 8：验证拓扑一致性                                                                                          
                                                                                                                  
  gmx grompp -f em.mdp -c complex.gro -p topol.top -o em.tpr                                                      
                                                                                                                  
  如果没有报错，拓扑就准备好了。                                                                                  
                                                                                                                  
  Step 9：标准 MD 流程                                                                                            
                                                                                                                
  # 溶剂化                                                                                                        
  gmx editconf -f em.gro -o newbox.gro -bt dodecahedron -d 1.0                                                    
  gmx solvate -cp newbox.gro -cs spc216.gro -p topol.top -o solv.gro                                              
  gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral                               
                                                                                                                  
  # EM                                                                                                            
  gmx grompp -f em.mdp -c solv_ions.gro -p topol.top -o em.tpr                                                    
  gmx mdrun -v -deffnm em                                                                                         
                                                                                                                  
  # NVT 平衡 (加位置约束)                                                                                         
  gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr \                                             
    -n prot_dna_index.ndx                                                                                         
  gmx mdrun -deffnm nvt                                                                                           
                                                                                                                  
  # NPT 平衡                                                                                                      
  gmx grompp -f npt.mdp -c nvt.gro -t nvt.cpt -r nvt.gro \                                                        
    -p topol.top -o npt.tpr -n prot_dna_index.ndx                                                                 
  gmx mdrun -v -deffnm npt                                                                                        
                                                                                                                  
  # 生产 MD                                                                                                       
  gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md.tpr                                               
  gmx mdrun -v -deffnm md -nb gpu                                                                                 
                                                                                                                  
  MDP 参数要点                                                                                                    
                                                                                                                  
  em.mdp:                                                                                                         
  integrator = steep                                                                                            
  nsteps     = 50000                                                                                              
                                                                                                                  
  nvt.mdp (平衡):                                                                                                 
  define = -DPOSRES -DPOSRES_DNA                                                                                  
  integrator = md                                                                                                 
  dt = 0.002                                                                                                      
  nsteps = 50000  ; 100 ps                                                                                        
  tcoupl = V-rescale                                                                                              
  tc-grps = Protein_DNA Water_and_ions                                                                            
  ref_t = 310                                                                                                     
  pcoupl = no                                                                                                     
  constraints = h-bonds                                                                                           
                                                                                                                  
  md.mdp (生产):                                                                                                  
  integrator = md                                                                                                 
  dt = 0.002                                                                                                      
  nsteps = 100000000  ; 200 ns                                                                                    
  tcoupl = Nose-Hoover                                                                                            
  tc-grps = Protein_DNA Water_and_ions                                                                            
  ref_t = 310                                                                                                     
  pcoupl = Parrinello-Rahman                                                                                      
  pcoupltype = isotropic                                                                                          
  ref_p = 1.0                                                                                                     
  tau_p = 5.0                                                                                                     
  constraints = h-bonds                                                                                           
                                                                                                                  
  ---                                                                                                             
  方案 3：MCPB.py + ParmEd 转换（最严谨）                                                                         
                                                                                                                  
  如果你希望 Zn 配位的参数不是手动估计，而是通过量子化学计算得到，走这条路线。                                    
                                                                                                                  
  全流程                                                                                                          
                                                                                                                  
  Phase 1：在 AMBER 中用 MCPB.py 生成参数                                                                         
                                                                                                                
  # Step 1: 准备 PDB (需要单独提取每个锌指 + Zn)                                                                  
  # Step 2: 用 tleap 加载 AMBER 力场 + ZAFF                                                                       
  tleap -f ZAFF/leaprc.zaff                                                                                       
  source leaprc.protein.ff14SB                                                                                    
  source leaprc.DNA.OL15                                                                                          
  mol = loadpdb ctcf_zf1.pdb                                                                                      
  # ... 每个锌指单独处理                                                                                          
  saveamberparm mol ctcf_zf1.prmtop ctcf_zf1.inpcrd                                                               
  quit                                                                                                            
                                                                                                                  
  # Step 3: MCPB.py 生成 Zn 配位参数                                                                              
  MCPB.py -i ctcf_zf1.pdb -s 1 -s 2  # 半经验方法                                                                 
  # 或使用 Gaussian 做 QM 优化:                                                                                   
  MCPB.py -i ctcf_zf1.pdb --qm_method B3LYP --basis_set 6-31G*                                                    
                                                                                                                  
  对 CTCF 的 11 个锌指，每个都需要单独跑 MCPB.py（因为每个 Zn 周围的配位环境略有不同）。                          
                                                                                                                  
  Phase 2：ParmEd 转换到 GROMACS                                                                                  
                                      
  parmed -p ctcf_zf1.prmtop -c ctcf_zf1.inpcrd --gromacs zf1.top zf1.gro                                          
                                                                                                                  
  对所有 11 个锌指生成的片段，然后手动组装成全蛋白的拓扑。                                                        
                                                                                                                  
  Phase 3：合并 + 标准 GROMACS 流程                                                                               
                                                                                                                
  将 11 个锌指片段的拓扑与 DNA 拓扑合并，后续同方案 2 的 Step 8-9。                                               
                                                                                                                
  优点： Zn 配位的 b0 和 kb 来自 QM 计算，发表级可信度。                                                          
  缺点： 11 个锌指 × MCPB.py = 工作量大，且需要 AMBER + Gaussian。                                              
                                                                                                                  
  ---                                                                                                           
  方案 4：CHARMM36 + specbond.dat 自动识别（最省力）                                                              
                                                                                                                  
  CHARMM36 力场自带的 specbond.dat 可以自动识别金属-配位残基之间的连接。如果 PDB 中 Zn
  的命名规范且距离合适，pdb2gmx 可以自动创建 Zn-Cys/His 的键。                                                    
                                      
  # 前提：PDB 中 Zn 在合理距离内 (< 2.5 Å)                                                                        
  # CYS 需要改名为 CYM，HIS 改为 HSD                                                                              
  # Zn 放在蛋白质链中（不是独立链）                                                                               
                                                                                                                  
  gmx pdb2gmx -f ctcf_prepared.pdb -o complex.gro -p topol.top \                                                  
    -ff charmm36 -water tip3p -ignh -ter                                                                          
                                                                                                                  
  specbond.dat 包含 Zn 的默认识别规则：                                                                           
  ZN    SG  CYS   0.26  0.23                                                                                      
  ZN    NE2 HIS   0.26  0.21                                                                                      
                                                                                                                  
  但 对 11 个 Zn 的体系，pdb2gmx 的自动识别经常失败或重复。你需要手动修改 specbond.dat 并测试。                   
                                                                                                                  
  为什么不推荐给 CTCF： pdb2gmx 对多个相同金属离子的去重逻辑会在 >1 个 Zn 时触发，11 个 Zn                        
  基本确定会出问题。                                                       
                                                                                                                  
  ---                                                                                                           
  方案 5：虚拟原子 (VSite) 方法                                                                                 
                                                                                                                  
  将 Zn 的质量分配到 4 个配位原子上，Zn 本身成为一个无质量的虚拟位点（virtual site），被约束在 4                
  个配位原子的几何中心。这种方法允许使用 5 fs 的大步长。                                                          
                                                                                                                
  缺点： 需要对 CHARMM36 的 aminoacids.rtp 进行修改，为每一个参与配位的 CYM 和 HSD 增加虚拟位点。工作量大，不适合 
  CTCF 的 11 个锌指。                 
                                                                                                                  
  ---                                                                                                             
  针对 CTCF 的特别注意事项                                                                                      
                                                                                                                  
  1. 11 个锌指的配位残基确定方法                                                                                
                                                                                                                  
  CTCF 的 11 个锌指与其经典 DNA 靶序列（如 CCCTC）结合。这是根据 UniProt 或文献确定每个锌指的 Cys/His             
  位置的最佳方法：                                                                                                
                                                                                                                  
  # 使用 MDAnalysis 或 BioPython 分析 PDB 中 Zn 到 Cys SG / His NE2 的距离                                        
  import MDAnalysis as mda                                                                                        
  u = mda.Universe("ctcf.pdb")                                                                                    
  zn = u.select_atoms("resname ZN")                                                                               
  for z in zn:                                                                                                    
      sg = u.select_atoms(f"around 3.0 (resname CYM and name SG)")                                                
      ne2 = u.select_atoms(f"around 3.0 (resname HSD and name NE2)")                                              
      print(f"{z} → SG: {sg.resids}, NE2: {ne2.resids}")                                                          
                                                                                                                  
  2. 配位残基质子化状态                                                                                           
                                                                                                                  
  ┌──────────────────────┬────────────────┬──────┬───────────────────────┐                                        
  │         残基         │      状态      │ 电荷 │         说明          │                                      
  ├──────────────────────┼────────────────┼──────┼───────────────────────┤                                        
  │ Cys (配位)           │ CYM (去质子化) │ -1   │ SH → S⁻ 才能配位 Zn²⁺ │                                      
  ├──────────────────────┼────────────────┼──────┼───────────────────────┤                                        
  │ Cys (非配位)         │ CYS            │ 0    │ 保持正常              │                                        
  ├──────────────────────┼────────────────┼──────┼───────────────────────┤                                        
  │ His (配位, 通过 NE2) │ HSD (H on ND1) │ 0    │ NE2 空闲配位 Zn²⁺     │                                        
  ├──────────────────────┼────────────────┼──────┼───────────────────────┤                                        
  │ His (配位, 通过 ND1) │ HSE (H on NE2) │ 0    │ ND1 空闲配位 Zn²⁺     │                                      
  ├──────────────────────┼────────────────┼──────┼───────────────────────┤                                        
  │ His (未配位)         │ HSD/HSE 按 pH  │ 0    │ 保持默认              │
  └──────────────────────┴────────────────┴──────┴───────────────────────┘                                        
                                                                                                                
  3. 体系规模                                                                                                     
                                                                                                                
  CTCF (727 aa) + 11 Zn + DNA（~20 bp ≈ 1300 原子）:                                                              
  - 总原子数 ~15000（蛋白）+ ~1300（DNA）+ 11（Zn）+ ~80000（水、离子）                                         
  - ~100,000 原子，适合 GPU 模拟                                                                                  
                                                                                                                
  4. 热力学温度                                                                                                   
                                                                                                                  
  锌指蛋白通常在 310 K 模拟。但注意 CTCF 的 DNA 结合特异性对温度敏感，建议先用 310 K，分析 RMSF 看锌指是否稳定。  
                                                                                                                  
  5. 是否保留 SCAN/KRAB 结构域                                                                                    
                                                                                                                
  CTCF 的 N 端有 ~220 aa 的非锌指区域（包含部分 dimerization 功能），C 端也有 ~90 aa 的非锌指区域。模拟建议：     
  - 只模拟锌指阵列（~ZF1-11, ~250-680 aa）                                          
  - 去除两端 intrinsically disordered 区域，减少体系大小                                                          
                                                                                                                
  ---                                                                                                             
  方案推荐总结                                                                                                  
                                                                                                                  
  ┌────────┬───────────────────────────────┬─────────────────────────────────────────┐                          
  │ 优先级 │             方案              │                  理由                   │                            
  ├────────┼───────────────────────────────┼─────────────────────────────────────────┤                            
  │ 首选   │ 方案 2（CHARMM36 + 手动成键） │ CTCF 仅需改配位列表 │                            
  ├────────┼───────────────────────────────┼─────────────────────────────────────────┤                            
  │ 次选   │ 方案 3（MCPB.py 转换）        │ 需要发表级精度时值得投入                │                            
  ├────────┼───────────────────────────────┼─────────────────────────────────────────┤                            
  │ 不推荐 │ 方案 1（非键）                │ 11 个 Zn 配位一定散开                   │                            
  ├────────┼───────────────────────────────┼─────────────────────────────────────────┤                            
  │ 可试   │ 方案 4（specbond）            │ 但 pdb2gmx 多 Zn 去重 bug 无法绕开      │                          
  └────────┴───────────────────────────────┴─────────────────────────────────────────┘                            

```

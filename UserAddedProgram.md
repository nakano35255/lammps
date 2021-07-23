# pair_style lj/cut/mod
This is made by modifying "pair_lj_cut.h/cpp" to use the extended Lennard-Jones potential
$
U(r) = 4 \epsilon \[(\sigma/r)^{12} - c (\sigma/r)^{6}\] (r < r_c)
$

## how to use
```sh
pair_style      lj/cut/mod ${rc}
pair_coeff      * * ${eps} ${sig} ${c}
```


# compute hexorder/atom/mod
This is made by modifying "compute_hexorder_atom.h/cpp".

There are several changes from the original "compute hexorder/atom".
(1) keyword "z" must be set. Only the atoms in the region (zlo < z <  zhi) are used to calculate $q_n$. The default value is zlo = zhi = 0.0. If the value is NULL, then $q_n$ is calculated as 0
(2) in the original "compute hexorder/atom", the value of $q_n$ is set to zero for atoms that have less than nnn neighbors within the distance cutoff. In contrast, "compute hexorder/atom/mod" calculate the value of $q_n$ even for atoms that have less than nnn neighbors within the distance cutoff, as 
$
q_n = \frac{1}{neighbors} \sum_{j=1}^{neighbors} e^{i n \theta}
$
If nnn < neighbors, then the nnn nearest atoms are used.

## how to use
```sh
compute         hexorder_mod all hexorder/atom/mod degree ${ndgree} nnn ${nnn} cutoff ${cutoff} z ${zlo} ${zhi}
```

Because the value of q_n is set to for atoms not in the specified compute group, it is desirable to use it with dynamic region.
```sh
region          dregion block 0.0 ${Lx} 0.0 ${Ly} ${zlo} ${zhi} units box
group           dgroup dynamic all region dregion
compute         hexorder_mod all hexorder/atom/mod degree ${ndgree} nnn ${nnn} cutoff ${cutoff} z ${zlo} ${zhi}
dump            1 dgroup custom 1 filename id c_hexorder_mod[*]
```
For the case in which the average for all atoms is needed, we do
```sh
region          dregion block 0.0 ${Lx} 0.0 ${Ly} ${zlo} ${zhi} units box
group           dgroup dynamic all region dregion
compute         hexorder_mod all hexorder/atom/mod degree ${ndgree} nnn ${nnn} cutoff ${cutoff} z ${zlo} ${zhi}
compute         hexorder_mod_ave dgroup reduce ave c_hexorder_mod[*]
fix             1 dgroup ave/time Nevery Nrepeat Nfreq c_hexorder_mod_ave[*] file filename
```
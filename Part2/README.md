



## Cloning the VSDBabySoC Repository:
Before cloning the repository, choose a directory where you wants to store your works.
```bash
cd ~/vcd/photos
```

> [!Note]
> I prefer storing my work in a `photos` folder under `~/vcd`, but you can also clone it into 'home' directory.

Clone the required sources from the [VSDBabySoC Repository](https://github.com/manili/VSDBabySoC):

```bash
git clone https://github.com/manili/VSDBabySoC.git
cd VSDBabySoC
ls
```
`ls` used for to list out the files inside the `VSDBabySoC folder` and verify its contents.

[]

---

### Analyse the contents of VSDBabySoC

After cloning the repository, it is important to understand the structure of the files and directories.
This helps in navigating the design, identifying modules, and preparing for simulation.

[]

---

### Verilog Source Files (*.v):
These files hold the RTL implementation of the BabySoC, defining its core modules and functional behavior.

`avsddac.v` – Implements the DAC (Digital-to-Analog Converter) module of the SoC.

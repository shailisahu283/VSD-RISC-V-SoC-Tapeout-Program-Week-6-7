# Week 6 — Sky130, OpenLANE & RTL2GDS (Detailed README)

> Course unit: Week 6 — Hands-on with Sky130 PDK, OpenLANE, floorplanning, cell design, timing analysis and RTL2GDS flow.

---

## Quick overview

This README is a companion to the Week 6 labs and lectures. It is structured to be both a learning guide and a practical lab reference. You'll find:

* Short, clear explanations of every subtopic listed for Week 6
* Practical lab steps, common commands, and troubleshooting notes
* Interactive checklists for each day and each lab
* A small Q&A section and special notes / facts for important topics
* A final submission checklist and a simple dashboard to track progress

Use this doc as the canonical Week 6 reference in your GitHub repository.

---

## Learning objectives (by the end of Week 6)

* Understand the Sky130 PDK and OpenLANE toolchain and how to run an RTL2GDS flow.
* Be able to perform basic floorplanning and placement tasks and reason about power distribution and CTS.
* Design a simple standard cell (inverter) in Magic, extract a SPICE netlist, and run ngspice characterization.
* Run timing analysis in OpenSTA with ideal and routed clocks and iterate on ECOs.
* Run TritonCTS and TritonRoute for clock tree synthesis and routing; check DRC and LVS basics.

---

## How to use this repo

1. Clone the Week 6 repo: `git clone <your-repo-url>`
2. Study the `DayX/` folders — each folder contains lecture notes, lab scripts, and a checklist.
3. Follow the checklists in order. Commit frequently and keep one branch per major lab.
4. Use the dashboard at the end of this README to mark completed tasks.

---

# Daily breakdown + explanations, labs, Q&A and special notes

> For each numbered item below you'll find: a short explanation, practical lab hints, and a small "special note / fact" where helpful.

---

## Sky130 Day 1 — Inception & context

### 1. Sky130 Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK

**What:** Origins and why open-source EDA matters; overview of the Sky130 PDK and OpenLANE ecosystem.
**Lab hint:** Read the PDK README, locate technology files under `skywater-pdk/sky130A/` and OpenLANE scripts under `openlane/`.
**Special fact:** SkyWater Technology donated Sky130 PDK to the open-source community — it enabled educational tapeouts.

### 2. How to talk to computers

**What:** Introduction to file formats and interfaces used in ASIC flow: RTL, Verilog, LEF/DEF, GDS, Liberty, SDC, SPICE.
**Lab hint:** Map each format to a flow stage: RTL ➜ synthesis (.v/.sv), LEF/DEF for physical, GDS for final mask.
**Special note:** Learning common CLI tools (make, git, docker, python scripts) dramatically speeds lab work.

### 3. Introduction to QFN-48 Package, chip, pads, core, die and IPs

**What:** Package vs die vs chip — QFN-48 is a package type; pads and IO ring vs core and standard cell area.
**Lab hint:** Visualize pad ring, keep-out, and core area when floorplanning.
**Special fact:** Package constraints can limit pin placement and I/O pad pitch which impacts floorplan.

### 4. Introduction to RISC-V

**What:** Brief intro to RISC-V ISA and why it’s popular in open-source silicon.
**Lab hint:** Use a small RISC-V core (e.g., Rocket or PicoRV32) as a practical RTL example for synthesis.
**Special note:** Simple RISC-V cores are perfect for end-to-end RTL2GDS experiments.

### 5. From Software Applications to Hardware

**What:** The path: Application ➜ Algorithm ➜ RTL ➜ Synthesis ➜ P&R ➜ GDS — mapping software behaviour to hardware blocks.
**Lab hint:** Identify which parts of a software algorithm map to combinational logic, sequential logic, memory blocks, or IP blocks.

### 6. SoC design and OpenLANE

**What:** Overview of SoC partitioning and how OpenLANE automates much of the RTL2GDS flow for digital designs.
**Special fact:** OpenLANE integrates multiple open-source tools (Yosys, OpenROAD, Magic, Netgen, TritonRoute, etc.).

### 7. Introduction to all components of open-source digital ASIC design

**What:** Short descriptions: Yosys (synthesis), OpenROAD (P&R), TritonCTS (CTS), TritonRoute (router), Magic (layout), OpenSTA (timing), ngspice.

### 8. Simplified RTL2GDS flow

**What:** High level steps: Design prep ➜ synthesis ➜ floorplan ➜ placement ➜ CTS ➜ routing ➜ DRC/LVS ➜ GDS.
**Interactive checklist:** `[] design-prep` `[] synthesis` `[] floorplan` `[] place & route` `[] DRC/LVS` `[] GDS`.

---

## Get familiar with open-source EDA tools

### 9. OpenLANE Directory structure in detail

**What:** Where `designs/`, `flow/`, `docker/`, `scripts/`, `configs/`, `tmp/` live. Know where `reports/` and `results/` are generated.
**Lab hint:** Inspect `flow.tcl`, `config.json`, and `designs/<my_design>/config.tcl`.

### 10. Design Preparation Step

**What:** Prepare netlist, constraints (SDC), liberty files, LEF (std cell LEF), tech LEF, and power/ground info.
**Commands:** `openlane` CLI or `./flow.tcl -design mydesign -step design_prep` (depends on your OpenLANE wrapper).
**Special note:** Missing or incorrect LEF/Lib files are common causes for later failures.

### 11. Review files after design prep and run synthesis

**What:** Check produced files: synthesized netlist, SDC, mapped cell list, reports. Run Yosys for synthesis.
**Lab hint:** Open the synthesis report (cells, area, timing) and make small fixes to SDC if needed.

### 12. OpenLANE Project Git Link Description

**What:** Keep a `README.md` per project explaining the config, versions, patches, and known issues.
**Special fact:** Reproducibility is crucial — include exact OpenLANE commit / docker image tag.

### 13. Steps to characterize synthesis results

**What:** Extract worst-case timing paths, area numbers, and gate counts — feed to floorplan decisions.
**Lab hint:** Use synthesis reports to decide target utilization and aspect ratio for floorplan.

---

## Sky130 Day 2 — Floorplanning and library cells

### 14. Utilization factor and aspect ratio

**What:** Utilization = ratio of cell area to core area; aspect ratio = core width/height.
**Rule of thumb:** 60–75% utilization is typical for standard cells; very high utilization increases routing congestion.

### 15. Concept of pre-placed cells

**What:** Pre-placed macros (memories, RAMs, IO pads) fix positions before placement so the placer respects them.
**Lab hint:** Mark macros in the DEF/LEF and check placer logs.

### 16. De-coupling capacitors

**What:** On-chip decoupling capacitors (or on-package caps) stabilize local power and reduce IR drop.
**Special fact:** At advanced nodes decaps may be implemented as top-level metal structures or dedicated macros.

### 17. Power planning

**What:** The power grid, straps, rails, and via planning to reduce IR drop and noise.
**Lab hint:** Plan rails early and generate route guides for power distribution.

### 18. Pin placement and logical cell placement blockage

**What:** Pin IO placement affects routing congestion; blockages prevent placer from placing cells in certain areas.

### 19. Steps to run floorplan using OpenLANE

**What / Commands:** Edit `config.json` or `floorplan.tcl` in design folder, then run the floorplan stage. Inspect logs & `floorplan.def`.

### 20. Review floorplan files and steps to view floorplan

**What:** Review LEF/DEF, floorplan GDS view, and Magic layout.
**Lab hint:** Use Magic to open the floorplan LEF/DEF or convert to a viewable format.

### 21. Review floorplan layout in Magic

**What:** Use Magic to visually inspect core, IO, block placements and check for obvious overcrowding.

---

## Library Binding and Placement

### 22. Netlist binding and initial place design

**What:** Map logical cells in netlist to physical library cells (binding) and perform initial placement.

### 23. Optimize placement using estimated wire-length and capacitance

**What:** Use estimated RC/wirelength models to guide placement optmizations.

### 24. Final placement optimization

**What:** Iteratively improve placement to reduce congestion and timing violations.

### 25. Need for libraries and characterization

**What:** Standard cell libraries + characterized Liberty (.lib) files: essential for timing and power analysis.

### 26. Congestion aware placement using RePlAce

**What:** RePlAce improves placement by modeling placement as electrostatic and reducing cell overlap and congestion.

---

## Cell design and characterization flows

### 27. Inputs for cell design flow

**What:** Transistor-level schematic, target drive strengths, LEF cell site and pin definitions, and tech files.

### 28. Circuit design step

**What:** Schematic capture at transistor level — decide W/L, stacking, and body ties.

### 29. Layout design step

**What:** Draw physical layout in Magic, obey design rules, create pins and power rails.

### 30. Typical characterization flow

**What:** Extract netlist (parasitics), run ngspice or Xyce corner simulations, produce liberty (.lib) timing and power numbers.

---

## General timing characterization parameters

### 31. Timing threshold definitions

**What:** Setup, hold, propagation delay, contamination delay, clock-to-Q, input/output delays.

### 32. Propagation delay and transition time

**What:** Propagation delay = time between input transition and output response; transition = output rise/fall time.

---

## Sky130 Day 3 — Cell layout & spice characterization

### 33. IO placer revision

**What:** Revisit IO pad placement to match package and signal integrity requirements.

### 34. SPICE deck creation for CMOS inverter

**What:** Create transistor-level SPICE netlist including devices, models, power rails, and measurement probes.

### 35. SPICE simulation lab for CMOS inverter

**Lab steps:**

1. Create netlist (`inv.spice`) with `.include sky130.lib`/.model files
2. Run `ngspice -b -o inv.log inv.spice`.
3. Plot node voltages and currents.

### 36. Switching Threshold Vm

**What:** The voltage where output equals input — important for noise margin.

### 37. Static and dynamic simulation of CMOS inverter

**What:** DC sweep for VTC, transient for switching and propagation/transition times.

### 38. Lab steps to git clone vsdstdcelldesign

**Lab hint:** `git clone <your-vsdstdcelldesign-repo>` — follow README to run Magic scripts and SPICE extraction steps.

---

## Inception of Layout – CMOS fabrication process

(Topics 39–46)

For 39–46, each step is a stage in CMOS fabrication. When designing layout, understand the physical representation of each processing step:

* 39 Create active regions — define diffusion areas for transistors.
* 40 Formation of N-well and P-well — implant regions for CMOS.
* 41 Formation of gate terminal — polysilicon gate layer.
* 42 LDD formation — reduces hot carrier effects.
* 43 Source – drain formation — implants and salicidation.
* 44 Local interconnect formation — first metal/connect layer.
* 45 Higher level metal formation — routing and power planes.
* 46 Lab introduction to Sky130 basic layers layout and LEF using inverter — map metal, poly, diffusion, implants to layout layers.

**Lab hint:** Use Magic technology file for sky130 and `techfile` to see layer names and purposes.

### 47. Lab steps to create std cell layout and extract spice netlist

**What / Steps:**

* Draw cell in Magic using proper site and pins
* Place power rails and taps
* Run DRC, then extract to generate `.spice` or `.sp` for ngspice

---

## Sky130 Tech File Labs (48–56)

### 48–49. Create final SPICE deck using Sky130 tech & characterize inverter

**What:** Use Sky130 models & spice include files, set corners (SS/TT/FF), run measurements for timing and power.

### 50–53. Magic tool options, DRC rules and loading Sky130 tech rules

**Lab hint:** `magic -d X11 -rcfile .magicrc <cell.mag>` — check `sky130A.magicrc` and tech files in the PDK.

### 54–56. Fix poly resistor spacing and other DRC issues

**What:** Many DRC issues are geometric — adjust spacing, enclosures or shape sizes to meet rules.
**Special note:** Keep good commit messages when you fix rules — this helps reviewers understand DRC fixes.

---

## Sky130 Day 4 — Pre-layout timing & clock tree

### 57–59. Convert grid info to track info, Magic layout to std cell LEF, include new cell in synthesis

**What:** Translating layout to LEF for place & route requires careful site and pin mapping.

### 60–62. Delay tables and their usage

**What:** Delay/Elmore/Liberty tables map input slopes and output loads to delay values. Understand the table axes (input transition, output load).

### 63–68. OpenSTA labs — post-synth timing

**What:** Configure OpenSTA with `.lib` files and SDC, run static timing analysis, check setup violations, run ECOs.

**Commands (example):**

```bash
openroad -noinit -exit -file run_sta.tcl
# or for OpenSTA direct usage: opensta -liberty mylib.lib -sdc constraints.sdc -verilog top.v
```

**Special note:** Setup and hold are both important — fixing one can worsen the other.

---

## Clock tree synthesis (CTS) TritonCTS and signal integrity (69–77)

### 69. H-tree algorithm and buffering

**What:** H-tree topology tries to equalize clock path lengths. TritonCTS automates buffer insertion and net splitting.

### 70. Crosstalk and clock net shielding

**What:** Clock nets are sensitive — use shielding and keep clock nets in separate routing layers when possible.

### 71–72. Lab steps to run CTS using TritonCTS & verify

**Lab hint:** Ensure CTS uses correct buffer library, buffer strength, and constraints. Verify sink delays and skew reports.

### 73–77. Timing analysis with real clocks

**What:** After CTS and routing, redo STA using routed net delays (SDF or SPEF). Analyze hold and setup with back-annotated delays.

---

## Sky130 Day 5 — Final RTL2GDS, routing & DRC

### 78–80. Maze routing & Lee’s algorithm and DRC

**What:** Maze/Lee’s algorithm is a classical grid-based pathfinder. Modern routers like TritonRoute implement more optimized techniques but understanding Lee’s algorithm helps learning.

### 81–83. Power Distribution Network and routing basics

**What:** Build a multi-tier PDN: straps at top metal, rails at mid metals, local vias to cell power pins.

### 84–87. TritonRoute features and output

**What:** TritonRoute follows route guides, manages inter-guide connectivity, and outputs final routed DEF/GDS and routing reports.

**Lab hint:** After routing, run DRC, LVS (if possible), and prepare final GDS for the tapeout submission.

---

# Labs & step-by-step checklists (condensed)

## Day 1 checklist (OpenLANE setup)

* [ ] Install / pull OpenLANE docker image (record version).
* [ ] Download Sky130 PDK (record commit/tag).
* [ ] Prepare `designs/<your_design>/` folder and place RTL, SDC, and constraints.
* [ ] Run `design_prep` and inspect outputs.

## Day 2 checklist (floorplan + placement)

* [ ] Read synthesis area & timing reports.
* [ ] Choose utilization & aspect ratio.
* [ ] Configure floorplan and run floorplan stage.
* [ ] Visual review in Magic.

## Day 3 checklist (cell layout + SPICE)

* [ ] Create inverter layout in Magic.
* [ ] Run DRC & extract netlist.
* [ ] Run ngspice characterization across corners.

## Day 4 checklist (timing & CTS)

* [ ] Convert cell LEF & include char libraries.
* [ ] Run OpenSTA for post-synth and post-CTS STA.
* [ ] Run TritonCTS and verify skew.

## Day 5 checklist (routing & DRC)

* [ ] Run TritonRoute and evaluate route guides.
* [ ] Run DRC and fix any violations.
* [ ] Generate final GDS and package files for submission.

---

# Common commands & example snippets

**OpenLANE (example)**

```bash
# From OpenLANE root
./flow.tcl -design my_design -tag sky130A -run_dir ./runs/my_design -stop_at floorplan
```

**Yosys (synthesis example)**

```bash
yosys -p "synth -top top; write_verilog synth.v; stat"
```

**ngspice**

```bash
ngspice -b -o inv.log inv.spice
```

**Magic**

```bash
magic -d X11 <cell.mag>
# run drc: in magic: dlist; drc check; extract
```

**OpenSTA (example tcl)**

```tcl
reset
read_liberty mycells.lib
read_sdc constraints.sdc
read_verilog synth.v
link_design top
report_timing
```

---

# Troubleshooting quick tips

* **Missing LEF/Lib errors:** Confirm correct paths and versions; add absolute paths if needed.
* **High congestion:** Lower target utilization by 5–10% and re-run placement.
* **DRC errors after route:** Read the DRC error geometry, open layout region in Magic and fix by changing spacing/width.
* **Setup violations:** Improve synthesis constraints (drive strengths), add buffering, or modify placement.

---

# Q&A (selective)

### Q: What is the minimal set of files to run OpenLANE for a simple design?

**A:** RTL top-level `.v`/`.sv`, SDC constraints, target std-cell LEF, liberty `.lib`, tech LEF (sky130 tech.lef), and a `config.json` or design config.

### Q: How to choose utilization?

**A:** Start with 55–65% for ease of routing; raise gradually if you need smaller area but watch for congestion.

### Q: When should I re-characterize my cell?

**A:** Any time you modify transistor sizes, stackings, or layout that affects parasitics — re-extract and re-run corner sims.

### Q: Why is clock shielding important?

**A:** Shielding reduces crosstalk that can introduce jitter and skew on clock nets — especially critical for high-speed designs.

---

# Special notes & facts (compact)

* Always record the exact toolchain versions and PDK commit used — this ensures reproducibility.
* Tapeout-ready flows require sign-off for DRC, LVS, and timing; educational flows may skip LVS but always run DRC.
* Power grid design is often the dominant factor in large designs — early attention saves painful ECOs later.


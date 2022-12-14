# QUAC-TRNG

This repository provides all necessary files and instructions to reproduce the results of [QUAC-TRNG](https://people.inf.ethz.ch/omutlu/pub/QUAC-TRNG-DRAM_isca21.pdf).

Please use the following citation to cite QUAC-TRNG if the repository is useful for you.

> Ataberk Olgun, Minesh Patel, A. Giray Yaglikci, Haocong Luo, Jeremie S. Kim, F. Nisa Bostanci, Nandita Vijaykumar, Oguz Ergin, and Onur Mutlu, "QUAC-TRNG: High-Throughput True Random Number Generation Using Quadruple Row Activation in Commodity DRAM Chips", ISCA'21.

[Link to QUAC-TRNG's extended version on arXiv](https://arxiv.org/pdf/2105.08955)

```
@inproceedings{olgun2022quactrng,
      title={QUAC-TRNG: High-Throughput True Random Number Generation Using Quadruple Row Activation in Commodity DRAM Chips}, 
      author={Ataberk Olgun, Minesh Patel, A. Giray Yaglikci, Haocong Luo, Jeremie S. Kim, F. Nisa Bostanci, Nandita Vijaykumar, Oguz Ergin, and Onur Mutlu},
      year={2021},
      booktitle={ISCA}
}
```

We implemented QUAC-TRNG using the open-source FPGA-based DRAM characterization infrastructure: [DRAM Bender](https://github.com/CMU-SAFARI/DRAM-Bender). To run the scripts provided in this repostiory, you need an FPGA setup with DRAM Bender installed.

Please check out the prerequisites and installation instructions in the [DRAM Bender](https://github.com/CMU-SAFARI/DRAM-Bender) repository.

## Talks

To learn more about QUAC-TRNG, please refer to the talks and slides below:

[Long Talk: SAFARI Live Seminar](https://www.youtube.com/watch?v=snvF3g3GfkI&list=PL5Q2soXY2Zi_tOTAYm--dYByNPL7JhwR9&index=6)  
SAFARI Live Seminar Slides ([pptx](https://people.inf.ethz.ch/omutlu/pub/QUAC-TRNG-DRAM_isca21-talk.pptx)) ([pdf](https://people.inf.ethz.ch/omutlu/pub/QUAC-TRNG-DRAM_isca21-talk.pdf))

[Short Talk: Conference Talk Video](https://youtu.be/J-S5ooZ_GYY)  
Conference Talk Slides ([pptx](https://people.inf.ethz.ch/omutlu/pub/QUAC-TRNG-DRAM_isca21-talk.pptx)) ([pdf](https://people.inf.ethz.ch/omutlu/pub/QUAC-TRNG-DRAM_isca21-talk.pptx))

## Installation Instructions

Assuming you have downloaded the DRAM Bender repository to `DBPATH`, copy everything provided in this directory to `DBPATH/sources/apps/QUAC-TRNG`. Then you can use the instructions below to reproduce QUAC-TRNG's results.

### Obtain NIST STS

Download the NIST standard test suite (NIST STS) for random number generators using [this link](https://github.com/arcetri/sts). Install it under `tools` using the instructions provided in the linked repository's README.

After installing NIST STS, `QUAC-TRNG/tools/sts` should point to the NIST STS executable.

### Copy SoftMC_reset to bin/

Run the following command to copy the reset program under bin/ directory.

`cp ../ResetBoard/SoftMC_reset bin/`

### Entropy Tests

To run entropy tests, simply execute the `run_ent.py` script (you have to execute this in QUAC-TRNG directory):

```
make -f Makefile.ent  
python3 scripts/run_ent.py <DIMM_NAME> <TEMPERATURE_IN_CELCIUS> <QUAC_ITERATIONS> <ROW_STRIDE>
```

Example: `python3 scripts/run_ent.py hytt04 50 1000 4`

This script will output one text file for every tested 4-bit data pattern (variable named placement in run_ent.py). Each line in the text file starts with the starting `row id` of a DRAM segment, followed by the observed entropy numbers for each bit in the segment.

### von Neumann Corrected Bitstreams

This will collect 1 Mb bitstreams from frequently switching bitlines and input them to NIST STS. The script will stop the moment it finds a cell that passes all NIST tests.

```
make -f Makefile.observe  
python3 scripts/run_vn_nist.py <DIMM_NAME> <TEMPERATURE_IN_CELCIUS>
```

Example: `python3 scripts/run_vn_nist.py hytt04 50`

### SHA-256 Bitstreams

This section describes how to get the NIST STS results for SHA-256 processed bitstreams. Doing so consists of two steps: 1) finding the DRAM segments with top 10 entropy, 2) generating bitstreams from the top-10-entropy segments and processing them using SHA-256 and performing NIST STS tests.

First, run `make -f Makefile.sha`.  

1- Obtain the entropy distribution to find the desired 4-bit data pattern (refer to "Entropy Tests").  
2- Get the segments with max. entropy: `python3 scripts/top_ents.py <DIMM_NAME> <PATTERN> <TEMPERATURE>`  
3- Run the SHA-256 bitstream generator script: `python3 scripts/sha_bitstreams.py <DIMM_NAME> <PATTERN> <TEMPERATURE>`

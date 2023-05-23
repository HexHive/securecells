---
title: Home
---

# SecureCells: A Secure Compartmentalized Architecture

<div class="intro-container">
  <div style="width: 100%">
    SecureCells is a novel virtual memory architecture for secure, efficient and flexible 
    compartmentalization.

    <!-- Secure  -->
    <div class="intro-container">
      <div class="flex-child center" style="width: 25%">
        <img height="50" src="{{ '/assets/svg/secure.svg' | relative_url }}">
      </div>
      <div class="flex-child">
        <b>Secure: </b>

        <li> Access control for code and data accesses         </li>
        <li> Secured call gates for inter-compartment calls    </li>
        <li> Checks on permission transfers                    </li>

      </div>
    </div>

    <!-- Performant  -->
    <div class="intro-container">
      <div class="flex-child center" style="width: 25%">
        <img height="50" src="{{ '/assets/svg/fast.svg' | relative_url }}">
      </div>
      <div class="flex-child">
        <b>Performant:     </b>

        <li> Range-based access control checks eliminates duplication in page-table permissions   </li>
        <li> Userspace instructions for fast compartment switch and permission transfers          </li>
        <li> Software operations for context isolation                                            </li>
      </div>
    </div>

    <!-- Flexible  -->
    <div class="intro-container">
      <div class="flex-child center" style="width: 25%">
        <img height="50" src="{{ '/assets/svg/flexible.svg' | relative_url }}">
      </div>
      <div class="flex-child">
        <b>Flexible:     </b>

        <li> 2-D permissions tables allows different policies                               </li>
        <li> Software operations can be omitted when unnecessary, improving performance     </li>
      </div>
    </div>




  </div>

  <div class="thumbnail center">
    <a href="https://infoscience.epfl.ch/record/301914">
    <img class="thumbnail" src="{{ '/assets/img/preprint.png' | relative_url }}">
    </a>
  </div>
</div>

# Publication
## Abstract

Modern programs are monolithic, combining code of varied provenance without isolation, all the while running on network-connected devices. A vulnerability in any component may compromise code and data of all other components. Compartmentalization separates programs into fault domains with limited policy-defined permissions, following the Principle of Least Privilege, preventing arbitrary interactions between components. Unfortunately, existing compartmentalization mechanisms target weak attacker models, incur high overheads, or overfit to specific use cases, precluding their general adoption. The need of the hour is a secure, performant, and flexible mechanism on which developers can reliably implement an arsenal of compartmentalized software. 

We present SecureCells, a novel architecture for intra-address space compartmentalization. SecureCells enforces per-Virtual Memory Area (VMA) permissions for secure and scalable access control, and introduces new userspace instructions for secure and fast compartment switching with hardware-enforced call gates and zero-copy permission transfers. SecureCells enables novel software mechanisms for call stack maintenance and register context isolation. In microbenchmarks, SecureCells switches compartments  in only 8 cycles on a 5-stage in-order processor, reducing cost by an order of magnitude compared to state-of-the-art. Consequently, SecureCells helps secure high-performance software such as an in-memory key-value store with negligible overhead of less than 3%.

## Paper Citation

SecureCells can be cited as per the BibTex citation below.

> (Note, this is currently a placeholder)

```
@inproceedings {Bhattacharyya:2023:SecureCells,
author = {Atri Bhattacharyya and Florian Hofhammer and Yuanlong Li and Siddharth Gupta and Andres Sanchez and Babak Falsafi and Mathias Payer},
title = {SecureCells: A Secure Compartmentalized Architecture},
booktitle = {44th IEEE Symposium on Security and Privacy (IEEE S&P 2023)},
year = {2023},
address = {San Francisco, CA},
url = {https://none},
publisher = {IEEE},
month = May,
}
```

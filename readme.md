# Pairing of Lecturers and Students for the ABB Tutored Dissertations

## Overview
This repository contains the code used to assign **tutors and examiners** to students for the **Applied Biosciences and Biotechnology (ABB) MSc tutored dissertations**.

## How It Works
The pairing process is handled by the notebook **`pairing.ipynb`**, which:

- **Inputs**:
  - Lecturer areas of expertise: **`lecturers.xlsx`**
  - Student topic preferences: **`students.xlsx`**
- **Outputs**:
  - A preliminary pairing: **`preliminary_pairing.csv`**
  - An optimized pairing: **`annealed_pairing.csv`**

The notebook **matches each student with a tutor and an examiner** by maximizing the **overlap in research interests**.

- **Each lecturer** is assigned at least **two topics** they can supervise.
- **Each student** selects **three topics** in order of preference.

## Matching Algorithm
The matching algorithm optimizes assignments using a **score function**, which considers:

1. **Student-tutor topic alignment**
2. **Overall alignment among student, tutor, and examiner**
3. **Tutor-examiner alignment** (to encourage familiarity in evaluation)
4. **Preferred role allocation for lecturers** (to balance workload and avoid consecutive-year tutoring)

The score function is:


$L = \sum_{\{s,t,e\}} \langle \vec \sigma_s, \vec \phi_t \rangle + \lambda \langle \vec \sigma_s,\vec \phi_t,\vec \phi_e \rangle + \beta \langle \vec \phi_t,\vec \phi_e \rangle + \eta (\tau_t + \epsilon_e)$

where the sum runs over all proposed **student-tutor-examiner triplets** $\{s,t,e\}$.

### Elements of the Score Function
- **Student Preferences $\vec \sigma_s$**: A vector encoding the student's ranked topic choices, where:
  - $\sigma_i = 1$ for the most preferred topic
  - $\sigma_i = -1$ for the least preferred topic
  - $\sum \sigma_i = 0$
- **Lecturer Topics$\vec \phi$**: A vector encoding lecturer expertise, e.g.,
  - $\vec \phi = (0,0,f,f,0,0,0,0,f)$, where **$f$** is normalized so that $||\vec \phi|| = \sum f^2 = 1$.
- **Constants**:
  - $\lambda > 0$**encourages triplets with overlapping interests**.
  - $\beta > 0$**promotes tutor-examiner topic alignment** (to avoid mismatched evaluations).
  - **Weights**$\tau$(tutor) and $\epsilon$(examiner) allow prioritization of roles for specific lecturers.
    - By default, $\tau = \epsilon = 1$for all lecturers.
    - Adjusting these values allows constraints, e.g., **to avoid a lecturer tutoring in consecutive years,** set $\tau = 0.1$for that lecturer.
  - The overall influence of **role preferences** is controlled via **$\eta$**.

### Optimization Strategy
1. **Greedy Initialization**:
   - Tutors and examiners are initially assigned based on **at least one shared topic of interest**.
2. **Simulated Annealing Refinement**:
   - The initial assignments are **iteratively optimized** to maximize the total score $L$.

# DecisionMakingExam
ExamProjectCognitiveScience

README - Decision Making FINAL (PlayAndPray)

A small Bayesian lantern for a public-goods night: this project tests whether group competitiveness predicts how groups cooperate and learn across rounds in a repeated contribution task, using a conditional cooperation / belief-updating model in JAGS. 
PlayAndPray
What this analysis does
The core question is framed as three directional hypotheses:
H1: Competitiveness → lower initial belief / baseline cooperation (βX_alpha < 0) 
PlayAndPray
H2: Competitiveness → stronger belief updating (βX_omega > 0) 
PlayAndPray
H3: Competitiveness → flatter conditional cooperation slope (βX_rho < 0) 
PlayAndPray
Where the model’s key latent parameters (per group) are:
α (alpha): initial expectation / baseline cooperation
ω (omega): learning rate (how fast beliefs update)
ρ (rho): conditional cooperation (how strongly behaviour follows the group) 
PlayAndPray
Files
PlayAndPray.rmd — source R Markdown for the analysis (data prep → plots → JAGS model → results).
PlayAndPray.html — knitted output (“Decision Making FINAL”, dated 2025-12-30). 
PlayAndPray
ExamData.xlsx — input dataset (contributions by round + covariates like Competitiveness). 
PlayAndPray
CC_competitiveness.jags — generated JAGS model file written by the script. 
PlayAndPray
Data expectations
The script assumes:
Individual-level rows have Group == 0 and a non-missing r1Contr (used to drop observers/non-players). 
PlayAndPray
Groups are separated by summary rows where Group == 1; the script reconstructs a GroupID via cumulative counting across these separators. 
PlayAndPray
Each reconstructed group is expected to have 4 players and 5 rounds (groupSize <- 4, ntrials <- 5). 
PlayAndPray
Contribution columns are named r1Contr … r5Contr and are reshaped long, then packed into an array c_arr[player, trial, group]. 
PlayAndPray
Method overview
Pre-processing
Reconstruct GroupID, then create a unique group index G for each Condition × GroupID.
Create a stable Player index (1..4) within each group to avoid player-overwriting bugs. 
PlayAndPray
Derived variables
Gga[t,g]: group mean contribution at trial t (including self). 
PlayAndPray
Xg: group mean Competitiveness, then standardised (X <- zscore(Xg)). 
PlayAndPray
A small switch vector isFirst avoids if(t==1) logic inside JAGS. 
PlayAndPray
Empirical sanity plots
Competitiveness vs total contributions (proxy)
Competitiveness vs mean round-1 contribution
A simple “robustness check” repeat plot 
PlayAndPray
Bayesian model (JAGS)
Belief update:
E[1,g] = alpha[g]
E[t,g] = E[t-1,g] + omega[g]*(Gga[t-1,g] - E[t-1,g]) 
PlayAndPray
Behavioural mean:
mu[t,g] = alpha[g] at t = 1
mu[t,g] = (1-rho)*E[t,g] + rho*Gga[t-1,g] for t > 1 
PlayAndPray
Contributions modelled as truncated normals on [0,10]. 
PlayAndPray
Regression of α, ω, ρ on group competitiveness X[g] via betaX_* parameters. 
PlayAndPray
Inference
Fit is run with jags.parallel (3 chains; 30,000 iters; 10,000 burn-in; thin 5; cluster 3). 
PlayAndPray
Output includes posterior summaries and directional posterior mass, e.g.
P(betaX_alpha < 0), P(betaX_omega > 0), P(betaX_rho < 0). 
PlayAndPray
Results (from the knitted HTML)
Posterior means (95% credible intervals):
betaX_alpha mean ≈ 0.969 (CI ≈ −2.08 to 6.74)
betaX_omega mean ≈ −0.323 (CI ≈ −4.31 to 2.96)
betaX_rho mean ≈ −0.320 (CI ≈ −4.08 to 2.51) 
PlayAndPray
Directional posterior support (posterior mass):
P(betaX_alpha < 0) = 0.323
P(betaX_omega > 0) = 0.443
P(betaX_rho < 0) = 0.5695 
PlayAndPray
Sceptical translation: the posteriors here look… unconvinced. Not a slam dunk for any hypothesis; H3 has the strongest directional mass, but it’s still hardly a marching band. 
PlayAndPray
How to run
Option A — Knit the R Markdown
Put ExamData.xlsx in a known location.
Important: the current script reads from an absolute path (/Users/.../ExamData.xlsx). You’ll likely want to change that to a relative path, e.g. data/ExamData.xlsx. 
PlayAndPray
Open PlayAndPray.rmd in RStudio and click Knit to HTML. 
PlayAndPray
Option B — Run chunks interactively
Execute the data-loading + preprocessing chunks first (they build c_arr, Gga_mat, X, isFirst).
Then run the JAGS section; it will write CC_competitiveness.jags and fit the model. 
PlayAndPray
Software / dependencies
You’ll need:
R + RStudio (recommended for knitting).
JAGS installed system-wide (so R can call it).
R packages typically implied by the code shown in the HTML:
readxl (for read_excel) 
PlayAndPray
dplyr, tidyr (for %>%, filter, mutate, pivot_longer, etc.) 
PlayAndPray
A JAGS interface providing jags.parallel (commonly R2jags). 
PlayAndPray
A zscore() helper (either defined elsewhere in your environment or from a helper script/package). 

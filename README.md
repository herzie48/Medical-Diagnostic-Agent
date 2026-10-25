# Medical Diagnostic Agent
 
An AI course project: one intelligent agent that interviews a patient, reasons about the answers, diagnoses, plans confirmation tests and proposes a safe prescription. Each step uses a different technique from the syllabus (CO1 to CO5).
 
> **Educational use only.** All data are synthetic and all probabilities are illustrative. This is not medical advice and must not be used for real health decisions.
 
## What it does
 
1. Asks the patient short yes/no questions, choosing the most informative one each time.
2. Updates its belief about 9 possible diseases after every answer.
3. Checks safety rules written in first-order logic and escalates emergencies.
4. Finds the cheapest test pathway to confirm the diagnosis.
5. Chooses a treatment strategy under uncertainty.
6. Builds a safe prescription that respects allergies, age, drug interactions and budget.
## Course outcomes covered
 
| CO | Topic | Where it is used | File |
|----|-------|------------------|------|
| CO1 | Agents, PEAS, environment types | The whole consultation is a percept, update, reason, act loop | `co1_agent.py`, `agent.py` |
| CO2 | Uninformed and informed search | BFS, DFS, UCS, Greedy and A* on the care pathway; next question chosen by information gain | `co2_search.py` |
| CO3 | Minimax, alpha-beta, CSP | Treatment game tree; prescription solver; age and ordering constraints | `co3_decision.py` |
| CO4 | First-order logic | Red flags, isolation and household risk using unification, forward and backward chaining | `co4_fol.py` |
| CO5 | Learning paradigms | Naive Bayes (supervised), k-means (unsupervised), Q-learning (reinforcement) | `co5_learning.py` |
 
## Project structure
 
```
medical_ai/
├── main.py            command-line entry point
├── agent.py           the agent loop that connects all modules
├── kb.py              symptom list and the simulated "real world" probabilities
├── co1_agent.py       PEAS, environment properties, agent types
├── co2_search.py      search algorithms on the care-pathway graph
├── co3_decision.py    minimax, alpha-beta, expectimax and the CSP
├── co4_fol.py         first-order logic engine
├── co5_learning.py    learning algorithms
├── GUIDE.md           explanation guide, demo script and viva questions
└── web/index.html     the same agent as an interactive web page
```
 
## How to run
 
### Python version
 
Requires Python 3.8 or newer. No packages to install.
 
```bash
cd medical_ai
 
python3 main.py --scenario pneumonia     # full agent on one patient
python3 main.py --scenario all           # all four example patients
python3 main.py --demo 2                 # one course outcome (1 to 5, or "all")
python3 main.py --interactive            # you answer the questions
 
python3 main.py --demo all > demo_output.txt   # save the output to a file
```
 
Example patients: `dengue`, `pneumonia`, `emergency`, `flu`.
 
### Web version
 
Open `web/index.html` in any modern browser. It needs an internet connection only for the fonts. The page has a chat on the left and the algorithms running live on the right, with every step tagged by course outcome.
 
## How the techniques fit together
 
```
Patient answers ──► CO1 agent loop
                      │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
 CO4 logic        CO5 learned       CO2 informed search
 red flags,       Naive Bayes       picks the next question
 household risk   belief            (information gain)
      └───────────────┼────────────────┘
                      ▼ diagnosis
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
 CO2 care path    CO3 game tree     CO3 prescription CSP
 BFS DFS UCS      minimax,          allergy, age, interaction,
 Greedy A*        alpha-beta        budget
```
 
## Constraints used
 
| Constraint | Kind |
|-----------|------|
| Age filter (migraine needs age 6 or more, strep throat 3 or more) | Domain filtering |
| Fever = NO implies high fever = NO | Constraint propagation |
| Ask about high fever only after fever = YES | Action precondition |
| At most 10 questions | Resource budget |
| Stop at 85% confidence | Goal test |
| A red-flag rule one answer from firing must be checked first | Safety constraint |
| Allergy, ulcer, dengue and age rules for drugs | CSP unary constraints |
| Azithromycin and ondansetron not together | CSP binary constraint |
| Total prescription cost at most 300 | CSP global constraint |
 
## Sample results
 
| Item | Result |
|------|--------|
| Cheapest path to confirm pneumonia | A* and UCS: cost 1100. BFS and Greedy: 1230 |
| Alpha-beta pruning | Same decision as minimax, 78 of 94 nodes visited |
| Prescription for a penicillin-allergic patient with an ulcer | Paracetamol + Azithromycin + Metoclopramide + ORS, cost 165 |
| Naive Bayes accuracy on test data (Python) | About 90% top-1, 99% top-3 |
| First-order rules versus propositional | 9 rules versus about 10,000 grounded rules for 4 people |
 
## Limitations
 
- The data are generated from the same probability tables the model learns, and symptoms are independent in them. This favours Naive Bayes, so its accuracy would be lower on real records.
- Only 9 diseases and 20 symptoms are modelled.
- Q-learning uses single-step episodes (a contextual bandit), not a full multi-step process.
- Drug and clinical rules are simplified for teaching.
- Nothing here has been clinically validated.
## Further reading
 
`GUIDE.md` has the detailed explanation of each module.

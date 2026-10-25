Medical Diagnostic Agent: Presentation and Explanation Guide

Educational demo for an AI course. The numbers are illustrative and the data are synthetic. This is not medical advice.

1. What the project is (30-second pitch)

A single intelligent agent interviews a patient, reasons about the answers, diagnoses, plans confirmation tests, and proposes a safe prescription. Each step is done by a different AI technique from the syllabus, so the demo covers CO1 to CO5 in one story.

                         ┌───────────────────────────────────────────────┐
  Patient answers ─────► │ CO1  AGENT LOOP (agent.py)                    │
  (percepts)             │  percept → update state → reason → act        │
                         └──────┬───────────┬───────────┬────────────────┘
                                │           │           │
            ┌───────────────────▼──┐  ┌─────▼──────┐  ┌─▼───────────────────┐
            │ CO4 First-order logic │  │ CO5 Learned │  │ CO2 Informed search │
            │ red flags, suspicions,│  │ Naive Bayes │  │ next best question  │
            │ household risk        │  │ belief      │  │ (info-gain heuristic)│
            └───────────────────────┘  └─────────────┘  └─────────────────────┘
                                │ diagnosis found
          ┌─────────────────────┼─────────────────────────┐
  ┌───────▼────────┐   ┌────────▼─────────┐   ┌───────────▼──────────┐
  │ CO2 Care path   │   │ CO3 Game tree     │   │ CO3 CSP prescription │
  │ BFS DFS UCS     │   │ minimax,          │   │ allergy, age, drug   │
  │ Greedy A*       │   │ alpha-beta,       │   │ interaction, budget  │
  └─────────────────┘   │ expectimax        │   └──────────────────────┘
                        └───────────────────┘
2. Course-outcome map (the slide to show first)
CO	Syllabus topic	Where it is used in the agent	File
CO1	Agents, PEAS, environment types, agent types, history, applications	Whole system is a goal-based + utility-based + learning agent	co1_agent.py, agent.py
CO2	Uninformed search (BFS, DFS, UCS) and informed search (Greedy, A*)	(a) cheapest test pathway to confirm a diagnosis, (b) choosing the next question greedily by information gain	co2_search.py, agent.py
CO3	Minimax, alpha-beta, CSP	(a) worst-case treatment choice, (b) safe prescription, (c) age and ordering constraints	co3_decision.py
CO4	FOL syntax, semantics, inference; propositional vs FOL	Red-flag rules, suspicion, isolation, household risk, with unification, forward and backward chaining	co4_fol.py
CO5	Supervised, unsupervised, reinforcement learning	Diagnostic model learned from data, patient clustering, treatment policy learned by Q-learning	co5_learning.py
3. How to run
cd medical_ai            # Python 3.8+, no packages to install
python3 main.py --demo 1        # CO1 ... also 2, 3, 4, 5, or "all"
python3 main.py --scenario dengue      # full agent: dengue | pneumonia | emergency | flu | all
python3 main.py --interactive          # let a classmate play the patient
4. CO1: Agent fundamentals

PEAS

	
Performance	correct diagnosis, few questions, low test cost, safe prescription, never miss an emergency
Environment	patient, hospital pathway, drug formulary
Actuators	ask, order tests, prescribe, escalate
Sensors	yes/no answers, age, allergies, history, household

Environment properties: partially observable (true disease is hidden), single agent, stochastic, sequential, semi-dynamic, discrete.

Agent type: model-based core (the evidence dictionary plus the FOL fact base is the internal state), goal-based (reach a confident diagnosis), utility-based (choose the treatment by utility), learning (Naive Bayes and Q-learning).

Agent program (see agent.run)

Percept: patient's answer.
Update state: evidence, FOL fact Has(patient, symptom).
Reason: forward chaining, Bayes update, constraint propagation.
Act: ask the next question, or stop and diagnose, or escalate an emergency.

History and applications (for the talk). Turing's 1950 paper "Computing Machinery and Intelligence" and the 1956 Dartmouth workshop start the field. MYCIN (Stanford, 1970s) was a rule-based expert system for diagnosing bacterial infections with certainty factors; it is the direct ancestor of this demo. Modern uses include medical imaging, triage chatbots and clinical decision support.

5. CO2: Search

Problem. States are places in the care pathway (Triage, Vitals, GP, CBC, NS1, Specialist, ...). Actions are "go to the next step". Step cost is test fee plus waiting cost. Goal is a *_Confirmed node.

Heuristic. h(n) is the cheapest remaining path ignoring waiting costs (a relaxed problem). It never overestimates, so it is admissible and consistent, which makes A* optimal.

Result (from the run)

Goal	Algorithm	Cost	Nodes expanded	Optimal?
Pneumonia	BFS	1230	6	no
	DFS	2210	7	no
	UCS	1100	8	yes
	Greedy	1230	3	no
	A*	1100	4	yes
Dengue	BFS	2760	7	no
	A*	1190	4	yes

Talking points

BFS minimises the number of steps, not the cost, so it picked a 3-step path that costs 2760.
Greedy was fast (3 nodes) but returned a suboptimal path, because it ignores the cost already paid.
A* matches UCS's optimal cost with about half the expansions.
Second use of informed search: at each turn the agent ranks every unanswered symptom by expected information gain (reduction in entropy of the diagnosis) and asks the best one. That is a greedy best-first choice with an information-gain heuristic.
6. CO3: Minimax, alpha-beta, CSP
6a. Treatment decision as a game tree
Level	Player	Choice
0	MAX (doctor)	Antibiotic / Antiviral / Supportive
1	MIN (nature)	True condition: Bacterial / Viral / Dengue
2	MAX (doctor)	Continue / Add supportive care / Hospitalize
3	MIN (nature)	Complication or not

Leaf = outcome score 0 to 100. The tree has 94 nodes.

Minimax gives Antibiotic 20, Antiviral 25, Supportive 40, so it picks Supportive: the best worst case.
Alpha-beta returns the same decision while visiting 78 of 94 nodes (16 pruned).
Expectimax replaces MIN nodes with probability averages using the Bayesian diagnosis. For the pneumonia patient (91% bacterial) it picks Antibiotic, which is different from minimax.

Why the difference matters: minimax is safe when you do not trust your probabilities; expectimax is better when the diagnosis is confident. Nature is not truly an adversary, so minimax here means "robust to the worst case".

6b. Prescription as a CSP
Part	In this demo
Variables	medication slots: fever_pain, infection, nausea, hydration
Domains	the drugs for each slot
Unary constraints	allergy (penicillin removes Amoxicillin), NSAIDs removed for gastric ulcer or dengue, aspirin removed under 16, antibiotic only for bacterial diagnoses
Binary constraint	Azithromycin and Ondansetron cannot be combined (QT prolongation)
Global constraint	total cost <= 300 (this removes the 200-cost IV fluids option)
Solver	backtracking + MRV (smallest domain first) + forward checking

Pneumonia patient (allergic, ulcer, nauseous): the allergy forces Azithromycin, which in turn forces Metoclopramide for nausea, giving a cheapest plan of Paracetamol + Azithromycin + Metoclopramide + ORS (cost 165). Plain backtracking tried 7 assignments, MRV with forward checking tried 4, and both gave the same answer.

6c. Other constraints used in the agent
Constraint	Type
Age filter (e.g. migraine needs age >= 6, strep throat >= 3)	domain filtering over diseases
fever = NO implies high_fever = NO	constraint propagation
Ask high_fever only after fever = YES	action precondition
At most 10 questions	resource budget
Stop when confidence >= 85%	goal test
Red-flag symptom pair one answer away from firing must be asked before concluding	safety constraint
7. CO4: First-order logic

Syntax used. Constants (alice, fever), variables (?p), predicates (Has, Suspects, LivesWith), universal quantification, definite clauses.

R1  ∀p  Has(p,fever) ∧ Has(p,cough) ∧ Has(p,body_ache)  ⇒  Suspects(p,influenza)
R7  ∀p,d  Suspects(p,d) ∧ Contagious(d)  ⇒  Isolate(p)
R8  ∀p,d,q  Suspects(p,d) ∧ Contagious(d) ∧ LivesWith(p,q)  ⇒  AtRisk(q,d)
R4  ∀p  Has(p,shortness_of_breath) ∧ Has(p,chest_pain)  ⇒  RedFlag(p,cardio_respiratory)
R6  ∀p,x  RedFlag(p,x)  ⇒  Emergency(p)

Inference implemented

Unification with occur check. Examples from the run: unify(Has(p,fever), Has(alice,fever)) = {p/alice}, unify(Knows(john,x), Knows(y,Mother(y))) = {y/john, x/Mother(john)}, and unify(x, f(x)) fails.
Forward chaining (Generalised Modus Ponens): from Has(tom,fever)... it derives Suspects(tom,influenza), then Isolate(tom) and AtRisk(meena,influenza).
Backward chaining (Prolog-style SLD resolution): ?- Isolate(who) answers tom; ?- Emergency(tom) is false.

Why FOL and not propositional logic? R8 says something about every pair of people. Propositional logic must write one rule per combination.

	FOL	Propositionalised
Rules in the KB	9	11,748 for just 4 people (22 constants)
Growth	constant	polynomial in the number of constants (grows as constants^variables)
1000 people	9 rules	over a billion ground rules
Same conclusions?	yes	yes (verified in the demo)

So both are sound and give the same answers, but lifted FOL inference with unification avoids the blow-up. (Typed variables would reduce the blow-up; the demo shows the untyped worst case.)

Resolution (for theory questions). To prove KB ⊨ α with full FOL: convert KB ∧ ¬α to CNF (eliminate ⇒, move ¬ inward, standardise variables, Skolemise, drop quantifiers), then resolve using unification until the empty clause appears. Backward chaining in this demo is the restricted Horn-clause version of that.

8. CO5: Learning paradigms

Data are synthetic: 3000 patients sampled from probability tables in kb.py (80% train, 20% test). The agent never reads those tables; it only uses what it learns.

Paradigm	Method	Result in the run
Supervised	Naive Bayes	89.8% top-1, 99.5% top-3
Supervised	Decision tree (ID3, entropy)	80.0%
Supervised	k-NN (k=7, Hamming)	82.3%
(baseline)	always predict most common disease	21.2%
Unsupervised	k-means, k=5, labels hidden	clusters like "diarrhea / abdominal pain / nausea" are 97% gastroenteritis; overall purity 52%
Reinforcement	Q-learning, ε-greedy, reward = noisy outcome	learns Antibiotic for bacterial, Antiviral for viral, Supportive for dengue, matching the best possible policy

Naive Bayes in one line: P(disease | symptoms) ∝ P(disease) · Π P(symptom | disease), with Laplace smoothing. The agent uses it to update belief after every answer.

Honest caveats to mention

Naive Bayes scores highest partly because the synthetic data are generated with conditionally independent symptoms, which is exactly its assumption. On real records it would be lower, and a proper comparison needs real data.
Q-learning here has single-step episodes (γ = 0), which is a contextual bandit. A full multi-step MDP of patient states would be the next extension.
9. Suggested 12-minute demo script
(1 min) Slide: pitch, architecture diagram, CO map.
(1 min) --demo 1: PEAS and agent types.
(2 min) --scenario pneumonia: walk through questions. Point at the [CO2], [CO4], [CO5], [CO3] tags as they appear.
(2 min) --demo 2: show BFS vs A* table; explain the relaxed-problem heuristic.
(2 min) --demo 3: alpha-beta pruning count, minimax vs expectimax, then the CSP removals.
(2 min) --demo 4: unification examples and the propositional blow-up table.
(1 min) --demo 5: accuracy table, clusters, Q-values.
(1 min) --scenario emergency: red flag stops the agent. Then limitations slide.
10. Likely viva questions
Question	Short answer
Why is this a goal-based agent?	It picks questions to reach the goal "confident diagnosis" and uses search and planning.
Why is A* optimal here?	h is admissible and consistent because it comes from a relaxed problem (waiting costs ignored).
Why did BFS give a costly path?	BFS is optimal only for equal step costs; here costs differ.
What does alpha-beta change?	Nothing in the answer, only fewer nodes (78 vs 94). Best case with good ordering is roughly the square root of the nodes.
Is minimax appropriate when nature is not an adversary?	It gives a worst-case-safe choice; expectimax is the probabilistic alternative and we show both.
What makes a CSP different from plain search?	State is a partial assignment, goal test is "all constraints satisfied", and constraints let us prune and propagate (MRV, forward checking).
Why does the FOL KB stay small?	Variables and unification stand for all individuals at once; propositional logic needs a copy per grounding.
Forward vs backward chaining?	Forward is data-driven (we have facts, derive everything). Backward is goal-driven (we have a query, find supporting facts).
Why is the Naive Bayes assumption unrealistic?	Symptoms are not independent given the disease (fever and high fever, for example); we handle that one case with a conditional parent.
Is this safe for real patients?	No. Synthetic data, illustrative numbers, tiny disease list, no validation. It is a teaching model.
11. File list
kb.py             shared symptom list and (simulated real-world) probability tables
co1_agent.py      PEAS, environment properties, agent types
co2_search.py     BFS, DFS, UCS, Greedy, A* on the care pathway
co3_decision.py   minimax, alpha-beta, expectimax, CSP prescription
co4_fol.py        FOL rules, unification, forward and backward chaining, grounding comparison
co5_learning.py   Naive Bayes, ID3, k-NN, k-means, Q-learning
agent.py          the agent loop that connects everything
main.py           command-line entry point

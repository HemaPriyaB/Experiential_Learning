
# TIDE-HF: Heart Failure Medication Titration System

A system that keeps an eye on heart failure patients and tells doctors when it is safe to increase medications — without waiting weeks for clinic appointments.

---

## The Real Problem

Heart failure patients need to be on four different medications. All four. At the right doses. But almost nobody gets there. Why? Because adjusting these drugs takes forever.

Here is how it usually goes: Patient gets out of the hospital. Doctor says take these four pills. But they have to start at low doses. You wait a week. Come back for a blood test. If everything looks okay, go up one dose. Wait another week. Blood test again. Maybe go up another dose. Repeat for months.

Most people cannot handle that. They go home, try to do it themselves, and by the time they get back to the doctor something bad has happened. They end up back in the hospital.

This project tries to fix that. Instead of waiting for clinic visits, have a system watching the patient's blood pressure and labs every single day. When things look stable, recommend going up a dose on the next drug. Keep going until they are on all four at full doses.

But here is the tricky part — you cannot just increase everything blindly. If someone's blood pressure drops too low, you cannot start an ACE inhibitor. If their potassium gets too high, you do not add the potassium-sparing drugs. So the system has to know all these rules and follow them strictly.

That is what we built.

---

## What We Actually Built

**The Data** — We used a real hospital database called MIMIC with thousands of actual patients. We pulled out the heart failure ones — about 12,000 patients. For each patient we grabbed everything: blood pressure readings, heart rate, potassium levels, kidney function, what medications they were on, what their diagnoses were. We organized it all so that each line represents one patient at one specific time.

**The Decision System** — Think of it like a checklist. For each patient, the system goes through seven steps. Step 1: Is this an emergency? Step 2: Does this patient have too much fluid or not enough? Step 3: What should we do with the water pills? And so on. At each step it checks the numbers, applies the rules a doctor would use, and makes a recommendation.

**The Code** — We wrote programs that take the messy hospital database and clean it up so the decision system can use it. Then we wrote the decision system itself. All of it is commented so you can read it and understand what is happening.

**The Explanation** — We wrote a whole guide explaining the project. We made presentations. We created a big poster you could hang on a wall. We explained everything in a way that a doctor could understand without being a programmer.

---

## Why This Actually Works

You can explain every decision. If the system says hold off on the blood pressure drug, you can ask why. The answer is: because their blood pressure is too low right now. That matters. You cannot have a black box making medication decisions.

Every rule we put in comes from actual published research. We did not make anything up. We just took what cardiologists already know and turned it into something a computer can do.

The system handles real messy data. Hospital data is not clean. Some patients are missing some lab values. Some readings are weird. The system does not crash. It just works with what it has.

It works on real patient data from thousands of people. We tested it and the recommendations make sense. When we looked at specific patients, the system would make calls that matched what an actual doctor would do.

---

## How to Actually Use This

You need to have Python installed on your computer. You also need access to the hospital database we used, which is publicly available but you have to sign up for it (free, just need to prove you understand privacy).

Once you have both of those, download this project:

```bash
git clone https://github.com/TeamPixelMinds/TIDE-HF.git
cd TIDE-HF
pip install -r requirements.txt
```

The system works in steps. Each step takes hospital data and adds something to it.

First, find the heart failure patients:
```bash
python k.py
```

Then pull out their vital signs:
```bash
python k3.py
```

Then pull their lab results:
```bash
python labs.py
```

Then get their medications:
```bash
python k4.py
```

Finally, combine everything into one clean dataset:
```bash
python k6.py
```

Now run the decision system on that dataset:
```bash
python run_logic.py hfref_final_dataset.csv
```

If you want to test it on just one patient to see what it does:

```python
from logic_engine import run_logic

patient_info = {
    'heart_rate': 105,
    'blood_pressure': 95,
    'oxygen': 93,
    'breathing_rate': 24,
    'kidney_function_creatinine': 1.8,
    'potassium': 4.2,
    'gfr': 45,
    'has_copd': False,
}

decision = run_logic(patient_info)
print(decision['diuretic_action'])  # Will say: INCREASE
print(decision['blood_pressure_drug_action'])  # Will say: HOLD
print(decision['why'])  # Explains the reasons
```

---

## What the System Actually Does

It goes through seven checks on each patient:

**Check 1 — Is This an Emergency?**

First, just check if anything is critically wrong right now. Is oxygen dangerously low? Is blood pressure bottomed out? Is potassium through the roof? Is kidney function failing? If something is seriously bad, stop. Do not touch the medications. Alert a doctor immediately.

**Check 2 — Too Much Fluid or Not Enough?**

Look at how fast the patient is breathing. If they are breathing fast, they probably have too much fluid in their lungs (we call that WET). If they are breathing slowly, probably not much fluid (DRY). Somewhere in the middle is normal. This one decision matters for pretty much everything else.

**Check 3 — Water Pills**

If they are WET, turn up the water pill dose. If they are DRY, turn it down. If kidney function is getting worse, be careful with it.

**Check 4 — Blood Pressure Drug**

This one has three conditions. Before you turn up the blood pressure drug (ACE inhibitor or similar), three things have to all be true: blood pressure has to be okay, potassium has to not be too high, and kidney function has to not be too bad. If any one of those is not okay, hold off.

**Check 5 — Heart Rate Drug**

Here is something that seems backwards but is true — you have to get the patient dried out before you give them the heart rate drug. If they still have too much fluid, skip it. Once they are dried out and heart rate is still high, then give it.

**Check 6 — New Drugs**

These are newer medicines that help protect the heart and kidneys. But you have to check kidney function first. You also have to check potassium because one of them affects potassium.

**Check 7 — Is It Getting Better or Worse?**

Look at heart rate and blood pressure together. If heart rate is going up and blood pressure is dropping at the same time, that is bad. Heart is struggling. Alert someone.

---

## The Data We Used

We used a real hospital database called MIMIC with actual ICU patients. It has everything a hospital tracks — heart rate, blood pressure, oxygen levels, blood tests, medications, diagnoses. Everything is time-stamped so you can see what happened when.

We filtered it down to just the heart failure patients. That gave us about 12,000 unique people. Some were in the hospital multiple times so we had 16,000 total hospital visits. For each visit we pulled all their data.

Then we had to clean it up because hospital data is messy. Some patients do not have certain blood tests. Some readings are duplicates. Times do not always line up perfectly. We fixed all that.

The final dataset has about 3 million lines of data. Each line is one patient at one specific hour with all their information at that time.

This is real data from real people. Not made up. Not perfect. But real.

---

## What We Learned

**Domain knowledge is not optional.** We spent the first two weeks just learning heart failure. What is ejection fraction? Why does potassium matter so much? Why is it so hard to titrate these drugs? You cannot build a safe medication system without actually understanding medication safety.

**Real data is messier than you think.** We wrote test cases in our heads. But real patient data has weird combinations of values you did not anticipate. Timestamps that do not line up. Missing information. You discover all of that when you actually run your code on real data.

**Working code on real data beats perfect planning.** We got something running on actual MIMIC data by week 8. That helped more than anything else. Suddenly we could see what was actually happening. The output made sense or did not make sense. We could fix things.

**Clinicians will want to know why.** Every decision needs to be explainable. The system says "do not start this drug" — a clinician will ask why. You need to be able to give the answer. That is not an afterthought. That is built in from the beginning.

---

## What Works Well

The core system does what it is supposed to do. You feed it patient data and it gives you medication decisions. The decisions are clinically sensible — when we looked at specific patients, the system made the calls we would expect a cardiologist to make.

The code is clean and documented. Someone could actually pick this up and understand it.

The decision logic is fully traceable. Every single rule comes from a clinical paper.

---

## What Needs Work

We used respiratory rate as a stand-in for fluid status. That is not perfect. The ideal would be a wearable patch that measures thoracic impedance. That would tell you exactly how much fluid is in the lungs. With just respiratory rate you are guessing a bit.

The SGLT2 data is limited. This drug was only approved in 2020 and the MIMIC dataset ends in 2022. So we only have two years of real data on these drugs. More history would help.

We built this on ICU data. But the real use case is after patients leave the hospital. Home monitoring. Post-discharge. ICU is a different context.

The trajectory logic is simplified. Right now we look at one reading and see if it is concerning. Ideally you would look at three readings together to see a trend.

---

## What Needs to Happen Next

**Fix the fluid measurement** — Right now we guess based on breathing. We need real wearable data that actually measures fluid in the lungs.

**Add prediction** — Can we predict which patients are about to get worse before it happens?

**Better trend tracking** — Look at patterns over time, not just one snapshot.

**Learn from results** — Compare what the system recommends vs what doctors actually did. Learn what works best.

**Hook it up to the hospital system** — Right now it is separate. Real use means connecting to the actual hospital computer system so doctors see the recommendations.

**Test with real patients** — Eventually this needs to be tested with actual post-discharge heart failure patients at home.

---

## The People Who Did This

Three of us worked on this. We divided up the work fairly evenly but everyone had areas where they focused more.

**Hema** learned all the heart failure stuff. Read the research. Figured out what the safety rules should be. Did a lot of the writing explaining everything.

**Jaya** designed how the whole system fits together. Wrote most of the code. Made the presentations and the diagrams that show how everything works.

**Srinivasa** handled getting the hospital database set up and connected. Wrote the code that pulls and combines all the data. Tested everything to find problems.

---

## What is in This Folder

**/code** — The actual programs. Everything you need to run it.

**/data** — Sample datasets so you can test without having to pull everything from the hospital database.

**/docs** — All the writing we did. How the system works. What we learned. The research we read.

**/notebooks** — Step-by-step guides showing how to run the code.

**/results** — Example output. Charts showing what the system recommended for different patients.

**/presentations** — Slides and posters we made. Flowcharts showing how the system works.

That is it. Pretty straightforward.

---

## How to Cite This

If you use this in your research or build on it, cite it like this:

```
Balaji, H. P., Galda, J., & Tummalapalli, S. R. (2026). 
TIDE-HF: Trajectory Integrated Decision Engine for Heart Failure. 
University at Buffalo.
https://github.com/TeamPixelMinds/TIDE-HF
```

---

## Data Access

To use MIMIC-IV you need to:

Go to physionet.org. Create an account. Complete their training module (it takes maybe an hour). Sign their data use agreement. Request access to MIMIC-IV through Google Cloud Platform. That is it. It is free.

The dataset is published and described here:
Johnson, A. E. W., et al. (2023). MIMIC-IV, a publicly available large intensive care unit database. Scientific Data 10, 1.

UB Box link for the Dataset - https://buffalo.app.box.com/folder/376612710501?s=r13g7syamcjolphph257jdxl897c0lap
You can find the data in folder named : CDA650<<Filtered_dataset<<Final_Dataset<<hfref_final_Dataset
---

## What We Actually Did

This was a 12-week masters course project. One class. Three of us. We had a client — Dr. Ionita who runs a healthcare company. He gave us the problem to solve and checked in with us every week to see how it was going.

First two weeks we learned what heart failure even is. Third and fourth week we explored the hospital database and the research. Fifth and sixth week we wrote the code to pull and clean the data. Seventh and eighth week we built the decision system. Ninth and tenth week we made presentations and wrote documentation. Last two weeks we cleaned everything up.

Every week we met with Dr. Ionita. He would point out issues, ask questions, suggest directions. That was really helpful. Way more helpful than us just working alone.

---

## What Actually Happened When We Tested It

We ran the system on 100 patients with tens of thousands 's of hourly readings. The system recommended:

- Emergency stop about 20% of the time — makes sense for ICU patients where things are often critical
- About half the patients had too much fluid — what you would expect with decompensated heart failure
- Increase water pills about 40% of the time — right amount for people with fluid overload
- Skip the heart rate drug on most patients — correct because most needed to be dried out first
- Recommend the newer drugs on about 70% — correct number for those without contraindications

We picked individual patients and walked through what the system recommended step by step. The recommendations made sense. The reasons made sense. It was not perfect but it worked.

---

## What Surprised Us

How much time data cleanup takes. We spent more time on the pipeline than on the logic engine. But that makes sense — garbage data in, garbage decisions out.

How messy real data is. You cannot just assume everything is documented cleanly or timestamps line up perfectly.

How much explainability matters in clinical AI. We could have written a neural network that probably would have been more accurate. But you could not ask it why. That is not acceptable when you are making medication decisions.

---

## License

MIT License. Use this however you want. Credit would be nice but not required.

---

## Contact

Questions? Issues? Want to contribute? Open an issue on GitHub or reach out.

Dr. Ciprian Ionita (the client) — ciprian.ionita@qas.ai

---

## Final Thought

There is a quote that summarizes this whole thing:

*"Nobody should wait two weeks to find out something went wrong."*

That is the problem. Patient leaves the hospital. Nothing is monitored. Two weeks later they come back because things have gotten bad. This system tries to change that. Continuous monitoring. Real-time feedback. Keeping patients safe at home.

We built something that works. It is not perfect. But it is a start.

---

Team Pixel Minds. University at Buffalo.

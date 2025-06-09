# FreshersFluCambridge

Group Production -> Social nature generation -> Timetable lectures meals -> budget creation -> calender finish -> disease spread -> disease analysis.
<br>
# AI SUMMARY BELOW
<br>
# Group Production:
1. Loading Data and Setting Up Students
First, the notebook loads in two key files: one with subject effects (like how likely a subject is to have female students or bigger groups), and another with the number of students per subject, college, and year.
For each student, it assigns:

A college and subject (from the data)
A year group (using rules, e.g. Engineering has 4 years, some subjects skip years)
A unique Student ID (just a running number)
Initial traits: gender (influenced by subject and college, e.g. Newnham is always female), and a group size (sampled from a normal distribution, usually between 4 and 12, then tweaked by subject).
For example, if a subject has a higher “group adaption size” in the traits file, students in that subject will tend to have larger friendship groups.

2. Forming Friendship Groups
The notebook then tries to put students into realistic friendship groups. Here’s how it works:

Most groups are single year and single college (because people are more likely to be friends with those in their year and college).
Group size is based on the “adapted group size” calculated earlier for each student.
The algorithm starts with an unassigned student and tries to fill their group by:
First, picking others from the same year and college.
If there aren’t enough, it relaxes the rules: first to same year (any college), then same college (any year), and finally anyone left.
There are also tweaks to encourage subject diversity (so not all groups are single-subject), and to allow more mixing in higher years.
Numbers that influence this:

Group size: Usually 4–12, but subject can add or subtract from this.
Gender: Probability from the traits file (e.g. some subjects are 70% female).
Year and college: Most groups are single-year, single-college, but not all.

3. Analyzing the Groups
After all students are assigned to groups, the notebook analyzes:

How many groups there are
Distribution of group sizes
How many groups are mixed-year, mixed-college, mixed-gender, or multi-subject
What percentage of groups are single-gender, single-subject, etc.
For example, it might report that 92% of groups are single-year, 85% are single-college, 18% are single-gender, and 6% are single-subject.

4. Optimizing the Groups
If the group structure isn’t diverse enough (e.g. too many single-gender or single-subject groups), the notebook runs an optimizer:

It tries to swap students between groups to increase diversity.
There are several steps, each with a target (like “get at least 5% of groups to be multi-college” or “reduce single-gender groups below 10%”).
The optimizer uses a few thousand swaps per step.

Numbers that influence this:

Targets: e.g. at least 3% multi-year groups, 5% multi-college, less than 10% single-gender, and average dominant subject proportion below 33%.
Swaps: Each step allows up to 10,000–20,000 swaps, but stops early if it gets stuck.
5. Saving the Results
Finally, the optimized group assignments are saved to a CSV file (students_group_final.csv) , ready for use in later analysis or simulation.

# Social Nature Generation

1. Loading Data
The notebook starts by loading two files:

students_groups_final.csv: This has all the students and their group assignments.
Subjecteffects.csv: This file contains, for each subject, a “baseline” value for how social or clubby students in that subject tend to be.
2. Preparing Subject Modifiers
The code cleans up the subject effects data and calculates two “modifiers” for each subject:
Social Modifier: How much more or less social students in this subject are compared to the baseline (which is 7).
Club Modifier: Same idea, but for clubbing propensity.
For example, if English has a baseline of 8 for social, its modifier is +1; if Engineering is 6, its modifier is -1.
3. Merging Data
The notebook merges the student data with the subject modifiers, so every student now has the right modifier for their subject.
4. Calculating Social and Club Propensity
For each student, the notebook calculates two scores:
Social Propensity (how likely they are to go to social events) and Club Propensity (how likely they are to go clubbing).
Both are calculated with a mix of randomness and rules:

Social Propensity:
Start with a random number between 3 and 8.
Add the subject’s social modifier.
Adjust for year group:
Year 1: +2
Year 2: +1
Year 3: -1
Postgrad: -2
Adjust for group size:
If group size > 5: +1
If group size < 3: -1
Clamp the final score between 1 and 10.
Club Propensity:
Start with a random number between 3 and 8.
Add the subject’s club modifier.
Adjust for year group:
Year 1: +3
Year 2: +1
Year 3: -2
Postgrad: -4
Adjust for group size:
If group size > 4: +1
If group size < 3: -1
Clamp the final score between 1 and 10.
This means:

First-years are much more likely to go clubbing (+3) and socialising (+2).
Postgrads are much less likely (up to -4 for clubbing).
Bigger groups make you more social/clubby, tiny groups less so.
The subject you study nudges your score up or down.
5. Saving the Results
The notebook drops the temporary modifier columns and saves the final data (with new propensities and a StudentID) to students_groups_social.csv

# Timetable lecture meals

1. Loading Data
The notebook starts by loading the student list, their subjects, and other relevant data (like year, college, group, etc.). It also loads a master list of all possible lectures, practicals, and meal times for each subject.

2. Assigning Lectures and Practicals
For each student, the code looks up their subject and year, and assigns them the correct set of lectures and practicals.
Each lecture/practical has a specific time slot (e.g., "T_123" for Tuesday at 3pm).
The timetable is built as a big table (DataFrame) with columns for every hour in the 4-week period (so 672 columns: T_0 to T_671).
For each student, the relevant slots are filled with the name of the lecture or practical; all other slots are marked as "Free".

3. Assigning Meal Times
The code then assigns meal times for each student.
Meal slots are usually at standard times (e.g., breakfast at 8am, lunch at 1pm, dinner at 6pm), but there’s some randomness to reflect real-life variation.
If a student has a lecture or practical during a standard meal time, the code tries to move the meal to the nearest free slot before or after.
Meal slots are marked as "Meal" in the timetable.

4. Handling Overlaps and Conflicts
If a student has a lecture and a meal at the same time, the code prioritizes the lecture and shifts the meal.
The code ensures that no two activities overlap for any student.

5. Saving the Timetable
The final timetable for all students, with all lectures, practicals, and meals assigned, is saved as a CSV file (e.g., student_schedules_with_meals.csv).
Each row is a student, each column is an hour, and each cell tells you what the student is doing at that hour (lecture, practical, meal, or free).

# Budget Creation

1. Loading and Merging Data
The notebook starts by loading two files:
students_groups_social.csv: Contains each student’s group, social, and club propensities, etc.
student_schedules_with_meals.csv: Contains each student’s full timetable (lectures, meals, etc.) for 4 weeks. These are merged together using the StudentID so that every student has all their info in one row.

2. Calculating Study Budgets
Every student starts with a base of 25 hours of study budget.
Week 1: All students get an extra 40 hours (since there are no lectures in week 1).
Weeks 2–4: For each week, the code counts how many hours are taken up by lectures (by counting all non-'Free' slots except meals), and subtracts that from 40. The leftover is added to the study budget for that week.
The total study budget is the sum of all these weekly calculations, ensuring no negative values.

3. Calculating Social Budgets
The base social budget depends on year:
First-years: 30 hours in week 1, 20 hours per week for weeks 2–4 (total 90).
Others: 20 hours in week 1, 15 hours per week for weeks 2–4 (total 65).
This base is then adjusted for each student’s “propensity” for social and club activities:
The adjustment is proportional to how much higher or lower their propensity is compared to the average, scaled by 25%.
For example, a student with much higher propensity than average gets a bigger social budget.
The final social budget is rounded and kept as an integer.

4. Calculating Outdoor Budgets
Students are sorted by their social budget (most social at the top).
They are divided into chunks of 50, and the order is shuffled within each chunk for randomness.
Outdoor budgets are assigned as follows (per week, then multiplied by 4 for 4 weeks):
10% of students: 1 hour/week
40%: 4 hours/week
40%: 8 hours/week
10%: 15 hours/week
This gives a realistic spread, with the least social students tending to get more outdoor time.

5. Calculating Sleep and Shopping Budgets
Shopping: Every student gets 3 hours per week, for a total of 12 hours over 4 weeks.
Sleep: Each student is assigned a random sleep requirement between 7 and 9 hours per night, multiplied by 28 days, and rounded to the nearest hour.

6. Saving the Final Budget File
The final DataFrame includes all original columns from both input files, plus the new budget columns (study, social, outdoor, shopping, sleep).
The columns are ordered so that all student info and timetable columns come first, followed by the budget columns.
The result is saved as schedules_with_final_budgets.csv.

NOTE THAT OUTDOOR, SLEEP AND SHOPPING BUDGETS ARE ARCHAIC AND NO LONGER USED IN SUBSEQUENT CODE!!!

# Calender Finish

1. Loading Budgets and Setting Up the Calendar
The notebook starts by loading the schedules_with_final_budgets2.csv file, which contains each student’s calculated budgets for social, study, sleep, outdoors, etc.
The calendar is set up as a giant DataFrame: each student has a row, and each hour in the first 4 weeks (672 hours) is a column (T_0 to T_671).

2. Allocating Social Activities
Each student’s SocialBudget is split across the 4 weeks using fixed fractions (1/3 in week 1, 2/9 in each of weeks 2–4).
For each week, the social budget is broken into “blocks” (typically 1–6 hours each, with some randomness).
The code tries to schedule these blocks in the evening (between 6pm and 4am), maximizing overlap with group members (so friends socialize together).
The result: every student gets a realistic set of social events, with timing and duration that vary by week and person.

3. Filling in Sleep
For each student, a nightly SleepRequirement (randomly 6–10 hours) is assigned.
Sleep is scheduled after the last social event or after a randomized bedtime (between 10pm and midnight).
If a student doesn’t get enough sleep at night, the code tries to add naps during the day.
This ensures everyone gets their required sleep, with realistic variation in bedtime and wake time.

4. Allocating Meals, Lectures, and Other Core Activities
Lectures and practicals are already in the timetable from earlier steps.
Meals are scheduled at standard times, but moved if they clash with lectures or other activities.
The code ensures no overlaps: each hour is only one activity.

5. Filling the Remaining Time: Supermarket, Alone, Outdoors, Library
Supermarket: Each student goes shopping 1–3 times per week, at random times between 7am and 11pm.
Alone (Admin): About 20% of remaining free time is spent on admin/life tasks.
Outdoors: 5–40% of remaining free time is spent outdoors, with the exact percentage drawn from a normal distribution (mean 25%).
Library: Any remaining free time is spent in the library.

6. Assigning Real-World Locations
Supermarket: 80% of students go to Sainsbury’s (Sidney Street), unless at Girton or Homerton, who go to their nearest supermarket. The other 20% go to the closest supermarket to their college.
Canteen: If coming from a lecture, 50% go to the subject’s nearest canteen, 50% to their college canteen. Otherwise, 95% eat at their college canteen, 5% at a random nearby canteen.
Library: 3rd/4th years have a 10% chance of using the University Library during the day; otherwise, students split between college and subject libraries, with some working in their own room or in a café.
Social: Social blocks are assigned to college bars, pubs, or clubs, depending on year and time of night. Groups stick together, and may “pub hop” or go clubbing after 11pm.

7. Final Outputs and Analysis
The final timetable for all students is saved as students_full_timetable_social.csv and ColocCotime_network_base.csv.

What You End Up With
A complete, hour-by-hour, location-aware timetable for every student for the first 4 weeks of term.
Every activity is assigned: lectures, meals, sleep, social, outdoors, library, supermarket, admin, and more.
Every “social” event is co-located for group members, and every location is a real place in Cambridge.
The result is a rich, realistic synthetic dataset that can be used for modeling things like disease spread, social networks, or student life.

# Disease Spread

1. Loading the Timetable and Setting Up Disease Tracking
The notebook loads the test_network_base.csv file, which contains the full, hour-by-hour, location-aware timetable for every student.
It initializes a disease state for every student at every hour (columns like Disease_T_0, Disease_T_1, ...), where each disease state is a string (e.g., '0' for susceptible, '1' for infected, '2' for recovered).

2. Defining Diseases
A Disease class is created, with parameters for transmission rate (chance of infection per contact per hour) and recovery rate (chance of recovery per hour).
Five diseases (A–E, and sometimes F) are defined, each with different transmission rates but the same recovery rate (e.g., 0.01 per hour, so average illness lasts about 4–5 days).

3. Initial Infection
At hour T=12, 20 students (about 1%) are randomly chosen and infected with all diseases (state '1' for each disease).
All other students start as susceptible ('0').

4. SIR Model Simulation
For each hour from T=12 to T=671 (the rest of the 4-week period), the notebook simulates disease spread using a SIR (Susceptible-Infected-Recovered) model:
For each disease, and for each location at that hour:
Students at the same location are considered “in contact.”
The chance of a susceptible student becoming infected depends on:
The number of infectious students present,
The disease’s transmission rate,
The contact rate for that location and hour (from Location_popularity.csv).
Infected students have a chance to recover each hour (based on recovery rate).
The disease state for each student is updated for the next hour.
Every infection event is logged (who infected whom, where, and when).

7. Output and Logging
The simulation produces:
Disease_track_completeV2.csv: The full disease state for every student, every hour, for all diseases.
infection_events_log.csv: A detailed log of every infection event (time, disease, source, target, location).
What You End Up With
A dynamic simulation of disease spread through a realistic student population, hour by hour, location by location.
The model captures:
How social structure, co-location, and activity patterns drive transmission,
The effect of different diseases’ infectiousness,
The full infection and recovery history for every student.
This dataset can be used to analyze outbreaks, test interventions, or study the impact of behavior and space on disease dynamics.

THIS PRODUCES A CSV FILE WTIH 15 MILLION ENTRIES!!!

# Disease Analysis

1. Load Simulation Results
The notebook loads the full disease simulation output (Disease_track_completeV2.csv) and the infection event log (infection_events_log.csv).
These files contain, for every student and every hour, their disease state for each disease, and a record of every infection event (who, when, where, and which disease).

2. SIR Curve Analysis
For each disease, the notebook calculates and plots:
The number of infected and recovered students at each hour.
These are shown as time series (epidemic curves), either separately for each disease or all together for comparison.
This helps visualize how fast each disease spreads and how quickly students recover.

3. Superspreader and Location Analysis
Superspreader Analysis:
Counts how many infections each student caused (as a source) for each disease.
Plots the distribution, highlighting the presence of superspreaders.
Location Analysis:
Counts how many infections occurred at each location for each disease.
Plots the distribution, identifying high-risk locations.

4. Infection and Recovery by Group
Calculates, for each subject and college:
The percentage of students infected at each hour (for each disease).
The percentage recovered at the end of the simulation.
Plots these as:
Line graphs (infection % over time by subject/college).
Heatmaps (subjects/colleges vs. time, colored by infection %).

5. Top Spreaders’ Demographics
Identifies the top 20% of students who caused the most infections (“superspreaders”) for a given disease.
Analyzes and plots their distribution by subject, college, and year group.

6. Recovery Analysis
Calculates and plots the proportion of students in each subject (or college) who have recovered from a disease by the end of the simulation.
What You Can Learn
Epidemic dynamics: How quickly and widely each disease spreads and recovers.
Group vulnerability: Which subjects or colleges are most affected.
Superspreader impact: How much a small number of individuals drive outbreaks.
Location risk: Which places are hotspots for transmission.
Demographic patterns: Which groups (by subject, college, year) are most likely to spread or recover from disease.

Ta dum!

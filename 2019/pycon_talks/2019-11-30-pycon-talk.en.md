# **Title**

Machine Learning Outside the Kaggle Lines

---

_Puns, jokes, or "hooks" in titles are okay, but make sure that if all someone knew was the title, they still would have some idea what the presentation is about._

_Please do not include any personally identifiable information. The initial round of reviews are anonymous, and this field will visible to reviewers._

# **Description**

Machine learning is more art than science, and it isn't alway clear how you go from Kaggle competitions to building your own machine learning project from scratch. You have a cool idea, but how do you turn it into an application that you can show off to your friends, basking in the warmth of their envy? More importantly, what mistakes do you need to avoid in order to keep the hope of such basking alive?

Come along and hear of my missteps, assumptions exposed and exploded, and the unpleasant surprises that come from doing machine learning in the wild. You'll learn about how to deal with haphazard data entry, depending on the kindness of strangers, boxes of different shapes and sizes, and, finally, what to do when all of this changes, because it will.

If you're interested in machine learning and are familiar with the basic tools and techniques, but are unsure of how to take a project idea to production, this talk will at least show you what not to do and how not to do it.

---

_Both your title and this description are made public and displayed in the conference program to help attendees decide whether they are interested in this presentation. Limit this description to a few concise paragraphs._

_Please do not include any personally identifiable information. The initial round of reviews are anonymous, and this field will visible to reviewers._

# **Who and Why (Audience)**

This talk is for people who are familiar with the popular tools of machine learning in Python, but have little experience with real-world machine learning projects or applications. This includes students, hobbyists, and professionals who are new to the industry. They know about, and probably have used, libraries such as Numpy, Pandas, Scikit-Learn, and Tensorflow, but mostly in the the context of a class project or Kaggle competition, where the problem is clearly defined, the provided data already cleaned and formatted, and the environment is either their own computer or someone else's responsibility.

From this talk, the audience will learn more about the challenges and pitfalls that come with building their own machine learning applications from scratch, so as to better avoid them and increase their chances of producing a complete, successful project.

---

_1–2 paragraphs that should answer three questions: (1) Who is this talk for? (2) What background knowledge or experience do you expect the audience to have? (3) What do you expect the audience to learn or do after watching the talk?_

_Committee note: The “Audience” section helps the program committee get a sense of whether your talk is geared more at novices or experienced individuals in a given subject. (We need a balance of both lower-level and advanced talks to make a great PyCon!) It also helps us evaluate the relevance of your talk to the Python community._

# **Outline**

1. Introduction (2 mins)
    * Introduce the overview of how Kaggle competitions are different from building an independent project.
    * Explain Aussie Rules Football, footy tipping, and inspiration for the project.
2. Defining the Problem (2 mins)
    * Understand all parts of the real-world problem you're trying to solve.
    * Translate business objectives into machine-learning objectives.
3. Understanding data sources (5 mins)
    * Unlike with Kaggle, you have to find and clean you're own data.
    * Data sources change over time and update on their own schedules.
    * Understanding sources can be as important as understanding the data itself.
4. Use all available resources (5 mins)
    * Read technical and domain-specific resources to better understand your objective and project.
    * Importing data from a package is less work than maintaining web scrapers.
    * It's good to share the work rather than go it alone.
4. Make data assumptions explicit (5 mins)
    * Data bugs are difficult to catch.
    * Add a lot of assertions about the shape and content of data to raise errors early.
    * With dynamic data sources, unexpected bugs pop up regularly; static unit tests won't catch them.
5. Maintainability matters (2 mins)
    * Kaggle code can be abandoned after a competition.
    * Maintainability matters for side projects, because this isn't something you turn in for the best possible score then forget about.
6. The Joy of Production (5 mins)
    * You don't need to deploy Kaggle models to the cloud.
    * Know your dependencies and what's available on your server.
    * Know your memory and CPU needs and what specs your server has.
7. Conclusion (2 mins)
    * First year: despite the mistakes, I won my office footy tipping competition by one whole point.
    * Second year: despite the improvements, I lost my office footy tipping competition by five points.
    * Doing things the right way improves your chances, but doesn't guarantee victory.

---

_The “outline” is a skeleton of your talk that is as detailed as possible, including rough timings or estimates for different sections. If requesting a 45 minute slot, please describe what content would appear in the 45 minute version but not a 30 minute version, either within the outline or in a paragraph at the end._

_Please do not include any personally identifiable information. The initial round of reviews are anonymous, and this field will visible to reviewers._

_Committee note: The outline is extremely important for the program committee to understand what the content and structure of your talk will be. The timings/percentages help us compare multiple talks that might have a similar abstract. We know that they are estimates and only capture your view at this moment in time and are likely to change before PyCon. We hope that writing the outline is helpful to you as well, to organize and clarify your thoughts for your talk! The outline will not be shared with conference attendees._

_If there’s too much to your topic to cover even in 45 minutes, you may wish to narrow it down. Alternatively, consider submitting a 3-hour PyCon tutorial instead. If you plan to do live coding during your talk, please describe your backup plan in case the live coding fails (for whatever reason). Suggestions include a pre-recorded video, or slides to replace the live coding._

# **Additional Notes**

* Code for the first season model: [FootyTipper](https://github.com/cfranklin11/footy-tipper)
* Code for the second season model:
    * Main application: [Tipresias](https://github.com/tipresias/tipresias)
    * Data processing and machine learning models: [Augury](https://github.com/tipresias/augury)
    * Data source: [BirdSigns](https://github.com/tipresias/bird-signs)
* Sample of related blog posts:
    * [Post](https://medium.com/@craigjfranklin/toward-a-better-footy-tipping-model-mistakes-were-made-ee5a6738741f) introducing the idea and model.
    * [Post](https://medium.com/@craigjfranklin/footy-tipping-with-machine-learning-models-assemble-5f884a7e8538) about model selection.
    * [Post](https://dev.to/englishcraig/footy-tipping-with-machine-learning-2019-season-review-55mc) summarising 2019 performance and plans for the future.

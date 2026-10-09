# How I Use AI in This Project

This project is maintained by Santiago Bentancur with OpenAI Codex as a collaborative tool. My goal is to learn by doing and share useful material—not to present AI-generated content as unquestioned expertise.

## What Codex helps with

I use Codex to discuss Terraform concepts, review explanations, suggest alternatives, and identify inconsistencies between the study guide and lab instructions. It also helps organize files, check links, review Terraform configurations, and draft documentation and original practice questions. When I request implementation help, it can make code or documentation changes for me to review.

These contributions are part of the project, not something I want to hide. They do not replace my responsibility for what I publish.

## What I decide

I choose the learning objectives, question explanations that do not make sense to me, and decide which suggestions to accept, change, or reject. I work through the labs, investigate Terraform's behavior, and use those observations to improve the exercises.

Using AI does not mean I already understand every topic in depth. The guide is also a record of what I am learning; I continue revisiting concepts and experimenting with the configurations.

## How I review changes

The process is iterative:

1. Identify a learning goal, unclear instruction, or technical problem.
2. Discuss possible explanations or changes with Codex.
3. Review the proposed result against the intended lesson and, where needed, official documentation.
4. Check the relevant behavior and revise anything that does not match the evidence.
5. Review the diff before including the change in the published project.

Different checks answer different questions:

- **Documentation review:** check topic placement, clarity, examples, and agreement between instructions and solutions.
- **Formatting and validation:** use `terraform fmt -check` and `terraform validate` to check style and internal consistency. These do not prove a deployment will succeed.
- **Behavior checks:** use `terraform console`, review plans, and perform disposable lab experiments when appropriate. An apply requires reviewing its proposed infrastructure changes and checking cleanup afterward.
- **Repository checks:** inspect links, file paths, and Git diffs, and keep credentials, state, private keys, and private study material out of commits.

Checks may be performed by me or with Codex assistance. A check passing is evidence for that specific check, not proof that the entire repository is correct or production-ready. I do not claim every example has been deployed after every edit.

## Sources, practice questions, and feedback

Official Terraform and provider documentation are the technical references used to resolve uncertainties. AI can suggest plausible but incorrect answers, so unsupported claims need investigation rather than confident repetition.

Public practice questions are drafted for this repository, not copied from paid courses or real certification exams. Private course-derived study material is excluded from Git. AI-assisted drafting and overlap checks do not establish legal clearance or guarantee technical correctness; human review still matters.

The [root README](../README.md#project-status) is the source of truth for project status. Feedback, corrections, and suggestions through issues or pull requests help me improve both the material and my understanding.

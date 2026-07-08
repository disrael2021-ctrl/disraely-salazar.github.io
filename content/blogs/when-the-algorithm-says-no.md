---
title: "When the Algorithm Says No: AI Ethics in Hiring and What Data Professionals Must Know"
date: 2026-07-07
draft: false
author: "Disraely Salazar"
tags: ["AI ethics", "algorithmic bias", "hiring", "data science", "machine learning"]
categories: ["blog"]
description: "From Amazon's scrapped hiring tool to a 2026 federal lawsuit still in court — AI screening tools are discriminating at scale. Here is what it means for data professionals."
image: /images/ai-ethics-slides/slide-01.jpg
---

The pitch sounds reasonable. Remove human bias from hiring. Let an algorithm assess candidates on objective criteria — skills, experience, track record — without the gut feelings, assumptions, and blind spots that humans bring to every decision. More fair. More efficient. More consistent.

It is a compelling argument. It is also, in several well-documented cases, exactly wrong.

Over the past decade, AI-powered hiring tools have discriminated against women, rejected Black applicants at disproportionate rates, penalized people with disabilities, and filtered out older workers — all while being marketed as neutral, data-driven, and objective. The problems are not isolated glitches. They are structural. And they raise questions that sit directly at the intersection of data science and ethics: What happens when the model is the problem? Who is responsible? And what does a data professional do when they are the one building or deploying the system?

<!--more-->

<div style="background:#EFF6FF;border-left:4px solid #0D6EFD;border-radius:6px;padding:1rem 1.25rem;margin:1.5rem 0;display:flex;align-items:center;gap:1rem;flex-wrap:wrap;">
  <div>
    <strong style="color:#1A1A3E;">Presentation available:</strong>
    <span style="color:#475569;"> This article has a companion slide deck covering the same analysis.</span>
  </div>
  <a href="/files/AIEthicsHiring_DisraelySalazar.pptx"
     download
     style="background:#0D6EFD;color:#fff;padding:0.45rem 1.1rem;border-radius:5px;text-decoration:none;font-size:0.9rem;font-weight:600;white-space:nowrap;">
    &#8659; Download Slides (.pptx)
  </a>
</div>

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-02.jpg" alt="The promise of AI hiring versus what it delivered" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
  <figcaption style="text-align:center;color:#64748b;font-size:0.85rem;margin-top:0.6rem;">What AI hiring tools promised — and what they actually delivered. Neutrality is not the default; it has to be built.</figcaption>
</figure>

---

## A Decade of the Same Failure

The cases covered here span 2018 to 2026. They involve different tools, different vendors, different companies, and different victims. But the pattern is identical every time: a system trained on historical data reproduces the biases embedded in that history. The tech changes. The failure does not.

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-03.jpg" alt="Timeline of three AI hiring discrimination cases: Amazon 2018, Workday 2023-2026, HireVue/Intuit 2025" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
  <figcaption style="text-align:center;color:#64748b;font-size:0.85rem;margin-top:0.6rem;">Three cases, one pattern — a decade of AI hiring failures rooted in the same design flaw.</figcaption>
</figure>

---

## The Case That Started the Conversation: Amazon, 2018

In 2018, Reuters reported that Amazon had quietly shut down an internal AI recruiting tool it had been developing since 2014. The tool was trained on ten years of resume data submitted to Amazon — data that reflected a decade of hiring decisions made predominantly by humans in an industry that was, and largely still is, male-dominated.

The model learned from that data. It learned that the resumes of people who had been hired at Amazon looked a certain way. And the resumes of women, statistically, looked different. The tool began penalizing resumes that included the word "women's" — as in "women's chess club" or "women's college." It downgraded graduates of all-women's colleges. Amazon engineers attempted to correct for these patterns but ultimately could not assure that the tool would not find other ways to discriminate. The project was abandoned.

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-04.jpg" alt="Amazon 2018 AI hiring case: what happened, the mechanism, the outcome" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
  <figcaption style="text-align:center;color:#64748b;font-size:0.85rem;margin-top:0.6rem;">The Amazon case broken down — what the tool did, why it did it, and why it could not be fixed.</figcaption>
</figure>

The lesson was stark: an AI system trained on historical data inherits the biases embedded in that history. If the past was discriminatory, the model will be too — unless someone actively engineers it not to be. Neutrality is not the default. It has to be built.

---

## Proxy Discrimination: The Technical Mechanism Behind the Bias

To understand why AI hiring tools fail in predictable ways, you need to understand proxy discrimination.

A proxy variable is a piece of data that is not itself a protected characteristic — race, gender, age, disability — but that correlates strongly with one. ZIP code, for example, correlates with race due to decades of residential segregation. University name correlates with socioeconomic background. Employment gap length correlates with caregiving responsibilities, which disproportionately affects women.

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-05.jpg" alt="Proxy discrimination: how neutral variables like ZIP code, university name, and employment gaps encode protected characteristics" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
  <figcaption style="text-align:center;color:#64748b;font-size:0.85rem;margin-top:0.6rem;">How facially neutral variables become proxies for protected characteristics — the core mechanism behind algorithmic discrimination.</figcaption>
</figure>

None of these are explicitly discriminatory inputs. But when a model learns that certain ZIP codes, certain universities, or certain career timelines predict worse performance on whatever outcome it is optimizing for, it encodes discrimination into its outputs — legally and ethically — regardless of whether race or gender was ever in the training data.

This is called **disparate impact**: when a facially neutral practice has a disproportionate negative effect on a protected group. It has been illegal under U.S. employment law since 1971. AI does not make disparate impact disappear. It makes it harder to detect and easier to deny.

---

## Mobley v. Workday: The Case Still in Court

The Amazon story is eight years old. If it were isolated, you might call it a lesson learned. It is not isolated.

In February 2023, Derek Mobley filed a federal lawsuit against Workday, Inc., one of the largest HR software platforms in the world. Mobley, a Black man over 40 with a degree from Morehouse College and professional experience across finance, IT, and customer service, had submitted more than 100 job applications through Workday's platform. He was rejected every time — often within minutes of submitting, sometimes in the middle of the night. He was never offered an interview.

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-06.jpg" alt="Mobley v. Workday case details and timeline: 2023 filing, 2026 rulings, active discovery" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
  <figcaption style="text-align:center;color:#64748b;font-size:0.85rem;margin-top:0.6rem;">The Mobley v. Workday case — still active in discovery as of mid-2026, with a potential class of thousands of plaintiffs.</figcaption>
</figure>

Mobley alleged that Workday's AI screening software had discriminated against him based on race, age, and disability. In March 2026, a federal court in California rejected Workday's strongest remaining attempt to have the case dismissed — ruling that the Age Discrimination in Employment Act applies to job applicants, not just current employees — and authorized notice to a potential class of thousands of additional plaintiffs over the age of 40 who experienced similar patterns of near-instant rejection.

As of mid-2026, the case is in active discovery. No trial date has been set.

What makes *Mobley v. Workday* significant is not just its scale, but its legal theory. The court has treated Workday — the software vendor, not the individual employers — as a potential legal "agent" of the hiring process. If that reasoning holds, it would mean AI vendors can be held liable for discriminatory outcomes produced by tools they built and sold. That is a structural shift in how algorithmic accountability is assigned.

---

## HireVue and Intuit: When the AI Cannot Hear You

In March 2025, the ACLU of Colorado filed a complaint with the Equal Employment Opportunity Commission on behalf of an Indigenous and deaf woman who had worked at Intuit for several years, receiving positive performance feedback. When she applied for an internal promotion, she was required to complete an AI-powered video interview through HireVue, a platform that analyzes candidates' facial expressions, word choice, and tone of voice.

She requested human-generated captioning as a reasonable accommodation. Intuit denied it. The AI evaluated her interview and generated feedback recommending that she "practice active listening."

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-07.jpg" alt="HireVue/Intuit case: what happened and the root failures in the AI system" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
  <figcaption style="text-align:center;color:#64748b;font-size:0.85rem;margin-top:0.6rem;">The ACLU complaint against HireVue and Intuit — four root failures that led to a qualified employee losing a promotion.</figcaption>
</figure>

The legal complaint alleges violations of the Americans with Disabilities Act, Title VII of the Civil Rights Act, and the Colorado Anti-Discrimination Act. It also raises a point that every data professional should sit with: the AI interview tool performs measurably worse when evaluating non-white applicants and deaf or hard-of-hearing speakers. The system was not tested adequately for accessibility. It was deployed anyway. A qualified employee lost a promotion as a result.

HireVue disputed the specifics of the complaint. But the underlying issue — that AI systems built on majority-group data will perform worse on minority-group users — is not disputed in the research literature. It is well-established, and has been for years.

---

## What This Means for Data Professionals

These are not stories about rogue technology or bad actors. They are stories about systems built by skilled data professionals, reviewed by legal and compliance teams, approved by executives, and deployed at scale — that nevertheless caused discriminatory outcomes.

That means data professionals are directly implicated.

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-08.jpg" alt="Four responsibilities of data professionals: building, evaluating, presenting results, and ethical escalation" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
  <figcaption style="text-align:center;color:#64748b;font-size:0.85rem;margin-top:0.6rem;">The four stages where data professionals carry direct ethical responsibility — from model design through deployment.</figcaption>
</figure>

If you are the analyst who built the model, you are responsible for interrogating the training data: Where did it come from? What historical decisions does it encode? What proxy variables are present, and do any of them correlate with protected characteristics?

If you are the analyst who evaluated the model, you are responsible for testing it across demographic groups — not just average accuracy, but disaggregated performance. A model that is 85% accurate on average but 60% accurate for Black applicants is not a neutral tool. It is a discriminatory one, dressed in the language of statistics.

If you are the analyst presenting the results, you are responsible for communicating what the model cannot do, not just what it can. The obligation to explain — to make model behavior legible to decision-makers — is an ethical requirement, not a nice-to-have. Black-box decision-making in high-stakes contexts like hiring is not just a technical limitation. It is a failure of accountability.

And if the organization deploys a system you have flagged as biased, the ethical framework developed in data science — and in courses like this one — gives you a clear path: document your concerns in writing, escalate through appropriate channels, and, if necessary, decline to continue working on a system you believe is causing harm.

---

## The Regulatory Environment Is Catching Up

Legislators have started to respond. Illinois' AI Video Interview Act requires employers to disclose when AI is used in video interview analysis and to obtain consent. New York City's Local Law 144, in effect since 2023, requires employers using automated employment decision tools to conduct annual bias audits and publish the results. California's new Automated Decision Systems regulations, effective October 2025, bring AI-driven hiring tools explicitly under the scope of its employment discrimination law.

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-09.jpg" alt="Regulatory landscape for AI hiring: NYC Local Law 144, California ADS Regulations, Illinois AI Video Interview Act, Mobley v. Workday precedent" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
  <figcaption style="text-align:center;color:#64748b;font-size:0.85rem;margin-top:0.6rem;">The regulatory landscape is closing in — four major developments reshaping legal accountability for AI hiring tools.</figcaption>
</figure>

The direction of travel is clear: the era of deploying AI hiring tools without transparency, without testing for bias, and without accountability is ending. The question is whether organizations will wait for litigation to force the change, or whether the data professionals inside those organizations will advocate for it earlier.

---

## Conclusion: Objectivity Is a Design Choice

The Amazon tool was not objective. The Workday algorithm was not neutral. The HireVue platform did not treat all candidates equally. In each case, the system reflected choices — about what data to train on, what outcome to optimize for, whose experience to center, and who to test it with.

Those are all design decisions. They are made by people. And they carry ethical weight.

The promise of AI in hiring is real: well-designed systems can reduce certain types of human bias, speed up processes that waste candidates' time, and surface qualified applicants who might otherwise be overlooked. But that promise is only redeemable if the people building and deploying these systems take seriously the obligation to ask who the model might harm — before it is deployed, not after the lawsuit is filed.

As data professionals, we are not just technical contributors to these systems. We are ethical actors within them. The cases above are a reminder that when the algorithm says no, someone decided what that algorithm would value. We should be honest about who that someone is — and what responsibility comes with it.

<figure style="margin:2rem 0;">
  <img src="/images/ai-ethics-slides/slide-10.jpg" alt="Conclusion: Objectivity Is a Design Choice" style="width:100%;border-radius:10px;box-shadow:0 4px 18px rgba(0,0,0,0.13);">
</figure>

---

*Disraely Salazar is a Data Analytics student at triOS College in Toronto, ON, with a background in business operations and supply chain management. This article was written as part of coursework in Ethics and Communications for Data Science.*

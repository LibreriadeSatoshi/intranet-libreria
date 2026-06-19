# 

# 

# Epic Teacher Onboarding — Refined version (intranet-first)

Consolidated working document. June 9, 2026\.

*Questions and suggestions:* 

*How to maintain consistency between Moodle data and application or wizard data* 

**Epic**

* [ENG-271 — Instructor onboarding & course submission (Intranet)](https://b4os.atlassian.net/browse/ENG-271)

**Stories**

* [ENG-272 — Story 0: Federated identity and transparent provisioning](https://b4os.atlassian.net/browse/ENG-272)  
* [ENG-273 — Story 1: "Teach on LdS" entry point](https://b4os.atlassian.net/browse/ENG-273)  
* [ENG-274 — Story 2: Course submission wizard](https://b4os.atlassian.net/browse/ENG-274)  
* [ENG-275 — Story 3: Save & resume drafts](https://b4os.atlassian.net/browse/ENG-275)  
* [ENG-276 — Story 4: Approval workflow and status communication](https://b4os.atlassian.net/browse/ENG-276)  
* [ENG-277 — Story 5: Handoff to Moodle, publication, and instructor profile](https://b4os.atlassian.net/browse/ENG-277)  
* [ENG-278 — Story 6: Instructor promotion tools](https://b4os.atlassian.net/browse/ENG-278)

## **Refinement context**

The original proposal modeled course submission as a hidden Moodle course, relying on the native "Course request" feature to collapse much of Stories 3–5 into configuration.

The approach is inverted: the flow lives **outside Moodle**, in a parallel intranet app (Librería de Satoshi's intranet). Moodle goes from being the container of the process to being the final destination of the content.

## **Architecture decisions (made — context)**

* The intranet is the **source of truth** for intake and approval. Moodle \= post-handoff execution.  
* The content contract between intranet and Moodle is a **Doc** (MKT\_BRIEFING / COURSE\_COURSE\_MASTER\_PLAN), not a YAML emitted by the intranet. The loader generates the YAML/spec internally from the COURSE\_MASTER\_PLAN using the **Claude API**.  
* Post-approval, the intranet does not modify course content. Usage analytics \= separate future flow, out of MVP scope.  
* Identity must be transparent.

**About moodle-course-loader (actual repo state, reviewed):**

* Python CLI that creates courses in Moodle via the Web Services API.  
* **Simple YAML path** (CourseSpec: template\_id, fullname, shortname, category\_id, summary, visible): only duplicates from a template (Phase 1).  
* **Plan ₿ path** (PlanBCourseBuilder \+ planb\_source): ALREADY builds complete structure in Moodle from a semi-structured markdown document (h1→Parts, h2→Chapters), creates sections and pages (create\_page, update\_section, numsections), uploads assets, renders HTML, embeds videos. **This machinery is reused for the COURSE\_MASTER\_PLAN.**  
* Current client: core\_course\_\* (duplicate, create, update, delete, get\_contents, get\_categories, get\_courses\_by\_field), upload\_file, set\_course\_image, update\_section, create\_page. **Does NOT yet have user/role/enrolment functions** (added by Story 101).

## EPIC — Instructor onboarding & course submission (Librería de Satoshi Intranet)

Build an intranet app, independent of Moodle, that is the source of truth for the course submission lifecycle: from a user deciding to teach, through a wizard, drafts, and an approval workflow managed by Operations, to the handoff to Moodle.

In the flow, the intranet generates typed documents (MKT\_BRIEFING for marketing, COURSE\_MASTER\_PLAN for the instructor). The loader processes the COURSE\_MASTER\_PLAN to build the course in Moodle. After the handoff, content lives in Moodle and the intranet keeps the record as history, the basis for future analytics. The intranet also hosts the public instructor profile and the course listing.

**MVP scope:** Story 0 \+ Stories 1–6. Platform/loader stories: 101, A, B, C.

### Story 0 \- Federated identity via dedicated IdP

*IdP \= Identity Provider*  
*OIDC \=  OpenID Connect. It is the identity layer built on top of OAuth 2.0.*

*sub \= user identifiers, which change between applications but the IdP manages as unique.*

**As** an aspiring teacher / staff member, **I want** to log into the intranet with a single account, **so that** I don't manage separate credentials and my work isn't interrupted if Moodle is down.

**Model (proposed):** 

* Dedicated IdP that Google and GitHub connect to. Intranet and Moodle are independent clients of the IdP via OIDC.   
* The IdP is the single source of identity for anyone entering through the intranet. Authentication (who you are) lives in the IdP; authorization (what you can do) lives in each system separately.

**Acceptance criteria**:

* **Intranet login methods (via IdP)**: user/password (local account in the IdP), Google (OIDC), or GitHub (OIDC), the user chooses. Nostr is not an intranet method; it lives only in Moodle (native). A teacher using Nostr in Moodle accesses the intranet with any of the three IdP methods, including a local account, without needing to bring an external social identity.  
* **Unique identity** — a single record (the IdP account); the intranet only stores the reference to the **IdP `sub`** (not the Google or GitHub `sub`). The three methods (local, Google, GitHub) can coexist in the same IdP account and resolve to the same `sub`.  
* **Failure independence:** If Moodle is down, the IdP and the intranet continue authenticating normally. Teacher/staff identity does not depend on Moodle.  
* **Provisioning on first login:** A new user entering through the landing page → IdP authenticates → the intranet creates their profile (instructor/staff) referenced by the IdP sub.

**Entry points (principle: the entry point decides the method):**

* Entry through the intranet (landing page "Become a teacher", or staff) → IdP (Google/GitHub).  
* Direct entry through Moodle (student to their course) → Moodle native login (Google/GitHub/Nostr/user-pass). Without touching the IdP.

**Single friction case, existing student → teacher:**

* Occurs on the landing page. The user already has a Moodle account (created as a student). When entering through the intranet via IdP, if the identifier matches (same Google/GitHub), it is linked. If not, there is a step to link their existing Moodle account to the IdP identity. Define the linking strategy and fallback.

**Scope, students outside the IdP for now:**

* Students remain in native Moodle. The IdP is only for teachers/staff. Expanding to students is a future phase (adding users to the same IdP, without redoing the architecture).

**Technical notes:**

* IdP: Authentik self-hosted (proportioned to the limited volume of teachers/staff). Only Google/GitHub (standard OAuth, out of the box), no Nostr-stage development, as Nostr remains outside the IdP.  
* The IdP OIDC token is for login. Calls to Moodle Web Services (create courses, assign roles, Story 101\) continue using wstoken. Authentication and API integration are separate planes.  
* Moodle 5.1: connect Moodle to the IdP as a core OAuth2 client (config, not plugin). 5.1 requires moving plugins to /public after upgrade, applies to Moodle config as a client.  
* Migration: map existing teachers/staff to the IdP sub (one-off, not permanent logic).

**Authorization (not managed by the IdP):**

* The IdP provides **identity** **and the broad role** (staff/instructor). Fine-grained authorization, what each person can do within the intranet or within Moodle, is managed by each system.  
* Intranet roles (own): instructor, reviewer/Ops, marketing.  
* Moodle roles (own, managed in Moodle): Student, Teacher, Manager, admin. The loader assigns Teacher upon publishing (Story 101); Manager and admin are managed in Moodle. The Moodle admin role is orthogonal to the intranet, the intranet neither reads nor controls it.

**Previous Spike Task** (recommended before implementing): validate Authentik for the teacher/staff volume, connect Moodle 5.1 as an OIDC client, confirm stable 1:1 sub, and the student→teacher linking strategy. (The spike is simplified compared to previous versions: no Nostr piece.)

## 

### Story 1 — "Teach on Librería de Satoshi" entry point

**As** a logged-in user, **I want** an easy & smooth way to start teaching, **so that** I can begin submission without hunting for it.

**Acceptance criteria:**

* Visible entry point — A "Become a Teacher" action in a consistent, documented location.  
* Routes to onboarding — When clicked, I land on the page that starts the Story 2 Teacher Profile & Course Creation & submission.  
* State-aware behavior — Not logged in → login (delegates to Story 0); logged in, no submission → new draft; has draft(s) → submissions dashboard; already an instructor → instructor dashboard.  
* Discoverable to the right people — Specify who sees it.

**Intranet ↔ Moodle role mapping (closed).**

* Intranet manages its own roles: instructor, reviewer/Ops, marketing.  
* Moodle manages its own: Student, Teacher, Manager, admin.  
* **reviewer/Ops** (intranet) → **Manager** (Moodle) for those who manage courses; some staff are **already admin** in Moodle. Manager and admin are managed **in Moodle**, not from the intranet.  
* **instructor** (intranet) → **Teacher / editing teacher** (Moodle), assigned by the loader on publish (Story 101).  
* **Student** (Moodle) — course students.  
* The Moodle **admin role is orthogonal** to the intranet: not read or managed from it.  
* Not needed for this flow: Course creator (creation done by the loader via API), Non-editing teacher, Guest.  
* Principle: the intranet authorizes intranet things; Moodle authorizes Moodle things. The IdP provides identity only, not authorization.

**Notes:**

* Intranet-native entry point (no longer Moodle's "Request a course").  
* With multiple drafts possible (Story 3), "has draft" leads to the dashboard, not to a single draft.  
* Authentication delegates to Story 0\.  
* Entry to the intranet is via the public landing on Librería's site ("Become a Teacher" CTA) → login (Story 0).

### 

### Story 2 — Teacher profile wizard

**As** an aspiring teacher, **I want** a step-by-step form to submit my profile, **so that** my profile will be easy and frictionless. 

**Scope**

This story covers:

* Creating a profile for users who aspire to become instructors/teachers on the platform.  
* Allowing applicants to continue to the next step of the teacher onboarding process.

This story does **not** cover:

* Course content information.  
* Collection of course metadata.  
* Generating a review-ready course package compatible with the GitHub review workflow defined in Story 4\.

**Required fields:**

* Name or Pseudo \- Public to Everyone \-   
* Email \- it should show the one is in our data base   
* Confirmation  Email \- it should show the one is in our data base \- Optional  
* Biography Short bio \- 50 to 300 words.  \- You can edit later   \-    
* GitHub username 

Social media section. This part is crucial for the students to get to know you

* Linkedin  \- Optional \-   
* Nostr \- Optional \-  
* X \- Optional \-   
* Years of teaching experience \- Optional \-  
* Indicate the environments where you have taught: (Multiple selection) (applies only if the field above is completed)   
- [ ] Online  
- [ ] In-Person  
- [ ] University Campuses  
- [ ] None of the above  
* Do you have an audience with whom you share your knowledge?  
* Where is your residency? We need this information to make payments.  
* Anything else you want to share with us \\: Block text 

### Story 3 – Course submission 

Udemy example ![][image1]

**As** a teacher, **I want** a step-by-step form to submit my course to get approved

With all this information, we will generated a .md file serves as the input artifact for Story 4, where the course structure and content breakdown will be created

**Scope**

This story covers:

* Collection of course metadata.  
* Allowing applicants to continue to the next step of the teacher onboarding process.

This story does **not** cover:

* Generating a review-ready course package compatible with the GitHub review workflow defined in Story 4\.

**Required Fields:** 

* Course Name  
* What type of course would you like to offer?  
- [ ] MOOC: Self-paced online course that students can take at any time.  
- [ ] Cohort-Based Course: Students progress together following a shared schedule and deadlines.

      

      IF Cohor-based:

      Selection process Option

* Option MOOC:  
              **Show the language Selection field**  
* Cohort Based:  
  		Show the next fields in the form:  
* Start date   
* End date  
* Classes \- How many classes does your course have  
* Class \- recurrence, helps validate that the start and     end dates are correct \- ( help text: our class not exceed 2+ hours class)  
* Time  
* Time Zone


* Language Selection field  
* Elevator pitch:   
  * What is it? Describe the course in 1-2 sentences:

If you are in the elevator for 30 seconds, you have to explain what the person will learn in 1-2 sentences.  Add resources about what an elevator pitch

* What problem is this course solving for students, if any?

Understanding the students' paint points  will allow you to create the perfect **benefits** for your students. 

* Write 3 Learning learning objectives, outcomes, competencies, or skills your students get after finishing your courses.  
* Your ideal student. Who's this for? Spell out the experience level, what they want to achieve, and why they need this course.  
* Do you have the ownership rights to the course content or is open source material?   
- [ ] I created it, and I have ownership rights to the course content   
- [ ] All the content is Open Source [\- Popular Strong Community \-](https://opensource.org/licenses?categories=popular-strong-community)    
- [ ] Do you agree that the course content may be shared under open-source content distribution licenses (Creative Commons, MIT, GNU GPL, etc.)?

Create the MKT Briefing in .md on Github \-\> The link is provided 

Upload video(s) to present your course on social media platforms. 

**Scope**

This story covers:

* Course content information.  
* Generating a review-ready course package compatible with the GitHub review workflow defined in Story 4\. Create a template with the information we have so far in a markdown on github, so the teacher can continue working on the course.

This story does **not** cover:

* The save process between the steps described in each stage, as this is defined in Story 3  
* The approval process covered in the Story.

**Required Fields:** 

- ### Class Number: **Title Class**

* Content Class  
  * Topics  
  * Objectives  
* Activities Class:   
  * Quiz   
  * Exercises  
* Source Materials

  # Course Name
  # What type of course would you like to offer?
- [ ] MOOC: Self-paced online course that students can take at any time
- [ ] Cohort-Based Course: Students progress together following a shared schedule and deadlines:

  | Field | Information |
  |---|---|
  | Start date | |
  | End date | |
  | Classes | How many classes your course has |
  | Time | |
  | Time Zone | |
---
  # Language
  # Write 3Learning learning objectives, outcomes, competencies, or skills your students get after finishing your courses.
      1.
      2.
      3.
  # Elevator pitch:  What is it? Describe the course in 1-2 sentences.
  If you are in the elevator for 30 seconds, you have to explain what the person will learn in 1-2 sentences.
  # What problem is this course solving for students, if any?


  # Content Course
  ## Class Number N: Title of Class N
  ## Content class
  ### Topics
  ### Objectives
  ## Activities Class
  - Quiz
  - Exercise
  - Etc...
  ## Source Materials

### Story 4—Save & resume drafts

**As** an instructor, **I want** my submission saved as a draft, **so that** I can finish it across multiple sessions.

**Acceptance criteria:**

* **Autosave per step** — "Next" or "Save draft" persist server-side against my account.  
* **Resume** — I return to the last incomplete step with prior steps pre-filled.  
* **Draft visible** — In my dashboard with status draft and a "Continue" action.  
* **No partial submission** — A draft does not enter review until explicit submission.  
* **Multiple drafts** — Several drafts of different courses simultaneously.  
* **Drafts listed** — The dashboard shows all of them, each with "Continue".

**Notes:**

* The draft is an intranet-owned entity (submission / course\_draft). It is NO LONGER a hidden Moodle course.  
* This is real development (persistence, model, "last incomplete step" logic), not configuration.  
* States are not deleted, they are recorded: basis for analytics (wizard abandonment). Don't build the analytics now, just don't destroy the trail.  
* Each draft is an independent row tied to the user.

### Story 5 — Course submissions reviewed via GitHub

**As** an instructor, **I want** to submit the final version of a course and get clear feedback, **so that** I know whether it's approved or needs changes. Additionally, **I want** the course to be automatically uploaded to Librería’s Moodle.

**Why GitHub:**

Teachers already know (or should know) Markdown and GitHub, and both are also well-known tools in the Bitcoin ecosystem. Instead of building a custom approval workflow, we reuse GitHub's Pull Request review process. Courses are written as Markdown files following the previously detailed document structure.

**Where:**

There is a single GitHub repository for courses (the "courses" repo). All submissions are Pull Requests against this repo. Each course lives in its own Markdown files/folder following the agreed structure.

**How it works:**

1. **Submit** — The instructor forks/branches the courses repo and opens a Pull Request adding their course as Markdown files.  
2. **Review** — A reviewer from **Ops** reviews the PR:  
   1. **Approve** the PR → the course is accepted.  
   2. **Request changes** → the instructor edits and pushes again.  
   3. **Close** the PR → the submission is rejected.  
3. **Changes loop** — Pushing new commits to the same PR re-opens it for review automatically. No new submission needed.  
4. **Accepted** — **Ops** merging the PR is the sign-off.  
5. **Publish** — On merge, a GitHub Action automatically uploads the course to Moodle in the **‘Hidden from Students’** state. No manual step needed.

**Acceptance criteria:**

* A course submission is a Pull Request against the courses repo, containing Markdown files in the required structure.  
* Status is what GitHub already shows: open, changes requested, approved, merged, or closed.  
* Reviewers must leave a comment when requesting changes or closing.  
* Every review action notifies the instructor (GitHub's built-in notifications).  
* A merged PR triggers a GitHub Action that uploads the course to Moodle.

**Notes:**

* Category, cohort, and short name can be defined as a companion .yml file, or in the markdown itself (TBD).  
* Authentication for PRs should be provided by GitHub  
* GPG signed commits would be a **very** nice addition in sync with how the Bitcoin ecosystem works.

### Story 6 — Handoff to Moodle, publication, and instructor profile

**As** an approved instructor, **I want** my course published and my profile marked as an instructor, **so that** students can find me.

**Acceptance criteria:**

* **Document generation** — On approval, generation of the MKT\_BRIEFING and the COURSE\_MASTER\_PLAN (Story A) is triggered following the temporal sequence (Story B).  
* **Handoff** — When the COURSE\_MASTER\_PLAN is ready, its reference enters the Sheet the loader processes; the loader builds the course (Story C).  
* **Handoff confirmation** — The intranet records the Moodle Link (write-back) and reflects the course as published.  
* **Instructor role (intranet)** — With ≥1 approved course, the profile shows the instructor indicator.  
* **Teacher role (Moodle)** — Assigned by the loader (Story 101).  
* **Course visible in its category** — After publishing, it sits in the category assigned by the reviewer.

**Notes:**

* Sub-state: approved (Ops ok) → published (loader confirmed, Moodle Link exists). The handoff is not instantaneous (loader is batch).  
* The "instructor marker" lives in the intranet (own data: ≥1 approved course).  
* The structure Google Doc is NO LONGER a manual bridge: the COURSE\_MASTER\_PLAN is the loader's input, which builds the structure automatically (Story C).

### Story 6 — Publish and course promotions [Aria Cediel](mailto:mkt@libreriadesatoshi.com)

*As a marketer working in Librería, I want every approved course to generate a ready-to-review social briefing immediately, and every published course to trigger the scheduled posting of that content, so promotion happens the same day the course goes live without depending on the loader's timing.*

Acceptance criteria:

1. **Trigger 1 — `approved`:** When the course status becomes `approved`, Aria receives a notification with the link to the MKT\_BRIEFING (per Story A/B).  
2. **Draft creation:** Aria reviews the briefing and creates the post drafts in Metricool (X, LinkedIn) — copy, image, hashtags — without scheduling a date yet.  
3. **Trigger 2 — `published`:** When the Moodle Link exists (course `published`, per Story 6 base), the Nostr draft is automatically updated with the live course link and scheduled for same-day posting.  
4. **Nostr exception:** Since Metricool doesn't support Nostr natively, this channel is posted manually by Aria; the briefing should flag it as a pending manual step.  
5. **Status tracking:** The Sheet/intranet reflects "promo ready" after step 2 and "promo published" after step 3\.

Notes:

* Decouples creative work (done at `approved`, with no time pressure) from the technical publish event (unpredictable due to batch loader).  
* Open question: posting cadence if multiple courses are approved/published the same day (proposal: max 1 new-course post/day, FIFO queue) — needs decision.

 

Story 7 — Instructor promotion tools [Aria Cediel](mailto:mkt@libreriadesatoshi.com)

*As an approved instructor, I want to receive a ready-to-share social kit when my course is published, so I can promote it on my own channels and attract students.*

**Proposed flow:**

1. **Base template:** the kit uses a pre-designed Canva template (horizontal format for X/LinkedIn \+ vertical format for IG Stories), with LdS branding.  
2. **Trigger — `published` (Moodle Link exists):** an email is sent to the instructor with the complete kit, since at this point the real course link exists.  
3. **Kit contents:**  
* Horizontal image (X/LinkedIn) — Canva link.  
* Vertical image (IG Stories) — Canva link.  
* 2-3 copy variants in Spanish, friendly tone, ready to copy/paste.  
* Direct link to the course on Moodle.

**Notes:**

* This email is independent from the `approved` email (which confirms approval but doesn't yet have the Moodle link).  
* Since the `approved → published` handoff is not instantaneous (batch loader, per base Story 6), the instructor may receive this second email with some delay relative to approval — worth communicating this so it doesn't cause confusion.

### Story 8 —

**Github \- Read mkt briefing approved**   
**Sent Canva API to create a digital asset**  
**Read/ download the asset**   
**Then asset and information should feed metricool as draft**   
**Ari/MKT responsible approves and posts . \-\>** 

Example when I signed [change.org](http://change.org) I can share my petition ![][image2]

**Acceptance criteria:**

* **Public instructor profile** — A shareable public page listing my approved courses.  
* **Social share** — Each approved course has shared actions (copy link \+ ≥1 network).  
* **Featured eligibility** — Admins flag courses as featured and surface them in a featured section.  
* **Marketing role in the intranet** — A marketing role exists with access to courses for promotion purposes. Fourth intranet role, alongside instructor, reviewer/Ops, and admin.  
* **MKT\_BRIEFING access** — Marketing consumes the generated MKT\_BRIEFING (Story A) as an intake input.  
* **Course data export** — Marketing can export each course's data (wizard fields: title, description, audience, requirements, curriculum, bio, code) in a structured format to feed their external tools.  
* **Markdown export per course** — Each course can be exported as a self-contained .md, a direct format for the team's content tools (fits the existing social content flow). Reuses data the intranet already owns; captures nothing new, just serializes.

**Notes:**

* No longer the custom-heavy story it was under the Moodle approach: public profile and featured are natural intranet features.  
* "Course page" \= intranet-native public landing (describes the course with wizard data) and links to the Moodle Link.  
* "Entity with two views" pattern: internal view (dashboard) and public view (landing). Same data, two renders.  
* "Featured" \= a featured flag on the course model, unrelated to Moodle.  
* Promotion is carried out by the marketing team and may require tools or data outputs to feed their own marketing tools — hence the marketing role in the intranet.

### Story 9 \- Doc reference

[Ivan Fuentes](mailto:ivan@libreriadesatoshi.com)define this story, and write reference documentation that supports all the work of teachers. Review [teach.udemy.com](http://teach.udemy.com)

Based on the levels described in the Marketing Briefing, display a help text indicating what type of student corresponds to each level.
---
title: Go Developer Survey 2020 Results
date: 2021-03-09
by:
- Alice Merrick
tags:
- survey
- community
summary: An analysis of the results from the 2020 Go Developer Survey.
---

## Thank you for the amazing response! {#thanks}

In 2020, we had another great turnout with 9,648 responses, about [as many as 2019](https://blog.golang.org/survey2019-results). Thank you for putting in the time to provide the community with these insights on your experiences using Go!

## New modular survey design {#new}

You may notice some questions have smaller sample sizes ("n=") than others. That's because some questions were shown to everyone while others were only shown to a random subset of respondents.

## Highlights {#highlights}

- Go usage is expanding in the workplace and enterprise with 76% of respondents using [Go at work](#TOC_4.1) and 66% saying [Go is critical to their company's success](#TOC_6.1).
- [Overall satisfaction](#TOC_6.) is high with 92% of respondents being satisfied using Go.
- The majority of [respondents felt productive](#TOC_6.2) in Go in less than 3 months, with 81% feeling very or extremely productive in Go.
- Respondents reported [upgrading promptly to the latest Go version](#TOC_7.), with 76% in the first 5 months.
- [Respondents using pkg.go.dev are more successful (91%)](#TOC_12.) at finding Go packages than non-users (82%).
- Go [modules adoption is nearly universal](#TOC_8.) with 77% satisfaction, but respondents also highlight a need for improved docs.
- Go continues to be heavily used for [APIs, CLIs, Web, DevOps & Data Processing](#TOC_7.).
- [Underrepresented groups](#TOC_12.1) tend to feel less welcome in the community.


## Who did we hear from? {#who}

Demographic questions help us distinguish which year-over-year differences may result from changes in who responded to the survey versus changes in sentiment or behavior. Because our demographics are similar to last year, we can be reasonably confident that other year-over-year changes aren't primarily due to demographic shifts.

For example, the distribution of organization sizes, developer experience, and industries remained about the same from 2019 to 2020.

<img src="survey2020/orgsize.svg" alt="Bar chart of organization size for 2019 to 2020 where the majority have fewer than 1000 employees" width="700"/>

<img src="survey2020/devex_yoy.svg" alt="Bar chart of years of professional experience for 2019 to 2020 with the majority having 3 to 10 years of experience" width="700"/>
<img src="survey2020/industry_yoy.svg" alt="Bar chart of organization industries for 2019 to 2020 with the majority in Technology" width="700"/>

Almost half (48%) of respondents have been using Go for less than two years. In 2020, we had fewer responses from those using Go for less than a year.

<img src="survey2020/goex_yoy.svg" alt="Bar chart of years of experience using Go" width="700"/>


<p id="TOC_4.1">Majorities said they use Go at work (76%) and outside of work (62%).
The percentage of respondents using Go at work has been trending up each year.</p>

<img src="survey2020/where_yoy.svg" alt="Bar chart where Go is being used at work or outside of work" width="700"/>

This year we introduced a new question on primary job responsibilities. We found that 70% of respondents’ primary responsibility is developing software and applications, but a significant minority (10%) are designing IT systems and architectures.

<img src="survey2020/job_responsibility.svg" alt="Primary job responsibilities" width="700"/>

As in prior years, we found that most respondents are not frequent contributors
to Go open-source projects, with 75% saying they do so "infrequently" or "never".

<img src="survey2020/foss_yoy.svg" alt="How often respondents contribute to open source projects written in Go from 2017 to 2020 where results remain about the same each year and only 7% contribute daily" width="700"/>

## Developer tools and practices {#devtools}

As in prior years, the vast majority of survey respondents reported working
with Go on Linux (63%) and macOS (55%) systems. The proportion of respondents who primarily develop on Linux appears to be slightly trending down over time.

<img src="survey2020/os_yoy.svg" alt="Primary operating system from 2017 to 2020" width="700"/>

For the first time, editor preferences appear to have stabilized: VS Code remains the most preferred editor (41%), with GoLand a strong second (35%). Together these editors made up 76% of responses, and other preferences did not continue to decrease as they had in previous years.

<img src="survey2020/editor_pref_yoy.svg" alt="Editor preferences from 2017 to 2020" width="700"/>

This year we asked respondents to prioritize improvements to their editor by how much they would hypothetically spend if they had 100 “GopherCoins” (a fictional currency). Code completion received the highest average number of GopherCoins per respondent. Half of respondents gave the top 4 features (code completion, navigating code, editor performance and refactoring) 10 or more coins.

<img src="survey2020/editor_improvements_means.svg" alt="Bar char of average number of GopherCoins spent per respondent" width="700"/>

A majority of respondents (63%) spend 10–30% of their time refactoring, suggesting that this is a common task and we want to investigate ways to improve it. It also explains why refactoring support was one of the most-funded editor improvements.

<img src="survey2020/refactor_time.svg" alt="Bar chart of time spent refactoring" width="700"/>

Last year we asked about specific developer techniques and found that almost 90% of respondents were using text logging for debugging, so this year we added a follow-up question to find out why. Results show that 43% use it because it allows them to use the same debugging strategy across different languages, and 42% prefer to use text logging over other debugging techniques.
However, 27% don't know how to get started with Go's debugging tools and 24% have never tried using Go's debugging tools, so there's an opportunity to improve the debugger tooling in terms of discoverability, usability and documentation.
Additionally, because a quarter of respondents have never tried using debugging tools, pain points may be underreported.

<img src="survey2020/why_printf.svg" alt="" width="700"/>


## Sentiments towards Go {#sentiments}

For the first time, this year we asked about overall satisfaction. 92% of respondents said they were very or somewhat satisfied using Go during the past year.

<img src="survey2020/csat.svg" alt="Bar chart of overall satisfaction on a 5 points scale from very dissatisfied to very satisfied" width="700"/>

This is the 3rd year we've asked the "Would you recommend…" [Net Promoter Score](https://en.wikipedia.org/wiki/Net_Promoter) (NPS) question. This year our NPS result is a 61 (68% "promoters" minus 6% "detractors"), statistically unchanged from 2019 and 2018.

<img src="survey2020/nps.svg" alt="Stacked bar chart of promoters, passives, and detractors" width="700"/>

<p id="TOC_6.1">Similar to previous years, 91% of respondents said they would prefer to use Go for their next new project. 89% said Go is working well for their team. This year we saw an increase in respondents who agreed that Go is critical to their company’s success from 59% in 2019 to 66% in 2020. Respondents working at organizations of 5,000 or more employees were less likely to agree (63%), while those at smaller organizations were more likely to agree (73%).</p>

<img src="survey2020/attitudes_yoy.svg" alt="Bar chart of agreement with statements I would prefer to use Go for my next project, Go is working well for me team, 89%, and Go is critical to my company's success"  width="700"/>

Like last year, we asked respondents to rate specific areas of Go development according to satisfaction and importance. Satisfaction with using cloud services, debugging, and using modules (areas that last year were highlighted as opportunities for improvement) increased while most importance scores remained about the same. We also introduced a couple new topics: API and Web frameworks. We see that web frameworks satisfaction is lower than other areas (64%). It wasn't as critically important to most current users (only 28% of respondents said it was very or critically important), but it could be a missing critical feature for would-be Go developers.

<img src="survey2020/feature_sat_yoy.svg" alt="Bar chart of satisfaction with aspects of Go from 2019 to 2020, showing highest satisfaction with build speed, reliability and using concurrency and lowest with web frameworks" width="700"/>

<p id="TOC_6.2">81% of respondents said they felt very or extremely productive using Go. Respondents at larger organizations were more likely to feel extremely productive than those at smaller organizations.</p>

<img src="survey2020/prod.svg" alt="Stacked bar chart of perceived productivity on 5 point scale from not all to extremely productive " width="700"/>

We’ve heard anecdotally that it’s easy to become productive quickly with Go. We asked respondents who felt at least slightly productive how long it took them to become productive. 93% said it took less than one year, with the majority feeling productive within 3 months.

<img src="survey2020/prod_time.svg" alt="Bar chart of length of time before feeling productive" width="700"/>

Although about the same as last year, the percentage of respondents who agreed with the statement "I feel welcome in the Go community" appears to be trending down over time, or at least not holding to the same upward trends as other areas.

We've also seen a significant year-over-year
increase in the proportion of respondents who feel Go’s project leadership
understands their needs (63%).

All of these results show a pattern of higher agreement correlated with
increased Go experience,
beginning at about two years.
In other words, the longer a respondent has been using Go,
the more likely they were to agree with each of these statements.

<img src="survey2020/attitudes_community_yoy.svg" alt="Bar chart showing agreement with statements I feel welcome in the Go community, I am confident in the Go leadership, I feel welcome to contribute, The Go project leadership understands my needs, and The process of contributing to the Go project is clear to me" width="700"/>

We asked an open text question on what we could do to make the Go community more welcoming and the most common recommendations (21%) were related to different forms of or improvements/additions to learning resources and documentation.

<img src="survey2020/more_welcoming.svg" alt="Bar chart of recommendations for improving the welcomeness of the Go community" width="700"/>


## Working with Go {#uses}

Building API/RPC services (74%) and CLIs (65%) remain the most common uses of Go. We don't see any significant changes from last year, when we introduced randomization into the ordering of options. (Prior to 2019, options towards the beginning of the list were disproportionately selected.)
We also broke this out by organization size and found that respondents use Go similarly at large enterprises or smaller organizations, although large orgs are a little less likely to use Go for web services returning HTML.

<img src="survey2020/app_yoy.svg" alt="Bar chart of Go use cases from 2019 to 2020 including API or RPC services, CLIs, frameworks, web services, automation, agents and daemons, data processing, GUIs, games and mobile apps" width="700"/>

This year we now have a better understanding of which kinds of software respondents write in Go at home versus at work. Although web services returning HTML is the 4th most common use case, this is due to non-work related use. More respondents use Go for automation/scripts, agents and daemons, and data processing for work than web services returning HTML.
A greater proportion of the least common uses (desktop/GUI apps, games, and mobile apps) are being written outside of work.

<img src="survey2020/app_context.svg" alt="Stacked bar charts of proportion of use case is at work, outside of work, or both " width="700"/>

Another new question asked how satisfied respondents were for each use case.
CLIs had the highest satisfaction, with 85% of respondents saying they were very, moderately or slightly satisfied using Go for CLIs.
Common uses for Go tended to have higher satisfaction scores, but satisfaction and popularity don’t perfectly correspond.
For example, agents and daemons has 2nd highest proportion of satisfaction but it’s 6th in usage.

<img src="survey2020/app_sat_bin.svg" alt="Bar chart of satisfaction with each use case" width="700"/>

Additional follow-up questions explored different use cases, for example, which platforms respondents target with their CLIs.
It's not surprising to see Linux (93%) and macOS (59%) highly represented, given the high developer use of Linux and macOS and high Linux cloud usage), but even Windows is targeted by almost a third of CLI developers.

<img src="survey2020/cli_platforms.svg" alt="Bar chart of platforms being targeted for CLIs" width="700"/>

A closer look at Go for data processing showed that Kafka is the only broadly adopted engine, but a majority of respondents said they use Go with a custom data processing engine.

<img src="survey2020/dpe.svg" alt="Bar chart of data processing engines used by those who use Go for data processing" width="700"/>

We also asked about larger areas in which respondents work with Go.
The most common area by far was web development (68%),
but other common areas included databases (46%), DevOps (42%)
network programming (41%) and systems programming (40%).

<img src="survey2020/domain_yoy.svg" alt="Bar chart of the kind of work where Go is being used" width="700"/>


<p id="TOC_7.1">Similar to last year, we found that 76% of respondents evaluate the current Go release for production use, but this year we refined our time scale and found that 60% begin evaluating a new version before or within 2 months of release.
This highlights the importance for platform-as-a-service providers to quickly support new stable releases of Go.</p>


<img src="survey2020/update_time.svg" alt="Bar chart of how soon respondents begin evaluating a new Go release" width="700"/>

## Modules {#modules}

This year we found near-universal adoption for Go modules, and a significant increase in the proportion of respondents who only use modules for package management. 96% of respondents said they were using modules for package management, up from 89% last year. 87% of respondents said they were using _only_ modules for package management, up from 71% last year.
Meanwhile, the use of other package management tools has decreased.

<img src="survey2020/modules_adoption_yoy.svg" alt="Bar chart of methods used for Go package management" width="700"/>

Satisfaction with modules also increased from last year. 77% of respondents said they were very, moderately or slightly satisfied with modules, compared to 68% in 2019.

<img src="survey2020/modules_sat_yoy.svg" alt="Stacked bar chart of satisfaction with using modules on a 7 point scale from very dissatisfied to very satisfied" width="700"/>

## Official documentation
Most respondents said they struggle with official documentation. 62% of respondents struggle to find enough information to fully implement a feature of their application and over a third have struggled to get started with something they haven’t done before.

<img src="survey2020/doc_struggles.svg" alt="Bar chart of struggles using official Go documentation" width="700"/>

The most problematic areas of official documentation were on using modules and CLI development, with 20% of respondents finding modules documentation slightly or not at all helpful, and 16% for documentation around CLI development.

<img src="survey2020/doc_helpfulness.svg" alt="Stacked bar charts on helpfulness of specific areas of documentation including using modules, CLI tool development, error handling, web service development, data access, concurrency and file input/output, rated on a 5 point scale from not at all to very helpful" width="700"/>

## Go in the clouds {#cloud}

Go was designed with modern distributed computing in mind,
and we want to continue to improve the developer experience of building
cloud services with Go.

- The three largest global cloud providers (Amazon Web Services,
Google Cloud Platform, and Microsoft Azure) continue to increase
in usage among survey respondents,
while most other providers are used by a smaller proportion of respondents each year.
Azure in particular had a significant increase from 7% to 12%.
- On-prem deployments to self-owned or company-owned servers continue to
decrease as the most common deployment targets.

<img src="survey2020/cloud_yoy.svg" alt="Bar chart of cloud providers used to deploy Go programs where AWS is the most common at 44%" width="700"/>

Respondents deploying to AWS and Azure saw increases in deploying to a managed Kubernetes platform, now at 40% and 54%, respectively.  Azure saw a significant drop in the proportion of users deploying Go programs to VMs and some growth in container usage from 18% to 25%.
Meanwhile, GCP (which already had a high proportion of respondents reporting managed Kubernetes use) saw some growth in deploying to serverless Cloud Run from 10% to 17%.

<img src="survey2020/cloud_services_yoy.svg" alt="Bar charts of proportion of services being used with each provider" width="700"/>

Overall, a majority of respondents were satisfied with using Go on all three
major cloud providers, and the figures are statistically unchanged from last year.
Respondents reported similar satisfaction levels with Go development for
AWS (82% satisfied) and GCP (80%).
Azure received a lower satisfaction score (58% satisfied),
and free-text responses often cited a need for improvements to Azure's Go SDK and Go support for Azure functions.

<img src="survey2020/cloud_csat.svg" alt="Stacked bar chart of satisfaction with using Go with AWS, GCP and Azure" width="700"/>

## Pain points {#pain}

The top reasons respondents say they are unable to use Go more remain working
on a project in another language (54%),
working on a team that prefers to use another language (34%),
and the lack of a critical feature in Go itself (26%).

This year we introduced a new option, “I already use Go everywhere I would like to,” so that respondents could opt out of making selections that don't prevent them from using Go. This significantly lowered the rate of selection of all other options, but did not change their relative ordering.
We also introduced an option for “Go lacks critical frameworks”.

If we look at only the respondents who selected reasons for not using Go, we can get a better idea of year-over-year trends. Working on an existing project in another language and project/team/lead preference for another language are decreasing over time.

<img src="survey2020/goblockers_yoy_sans_na.svg" alt="Bar charts of reasons preventing respondents from using Go more" width="700"/>

Among the 26% of respondents who said Go lacks language features they need,
88% selected generics as a critical missing feature.
Other critical missing features were better error handling (58%), null safety (44%), functional programming features(42%) and
a stronger / expanded type system (41%).

To be clear, these numbers are from the subset of respondents who said they
would be able to use Go more were it not missing one or more critical features they need,
not the entire population of survey respondents. To put that in perspective, 18% of respondents are prevented from using Go because of a lack of generics.

<img src="survey2020/missing_features.svg" alt="Bar chart of missing critical features" width="700"/>

The top challenge respondents reported when using Go was again Go's lack of generics (18%), while modules/package management and problems with learning curve/best practices/docs were both 13%.

<img src="survey2020/biggest_challenge.svg" alt="Bar chart of biggest challenges respondents face when using Go" width="700"/>

## The Go community {#community}

This year we asked respondents for their top 5 resources for answering their Go-related questions. Last year we only asked for top 3, so the results aren't directly comparable, however, StackOverflow remains the most popular resource at 65%.
Reading source code (57%) remains another popular resource while reliance on godoc.org (39%) has significantly decreased. The package discovery site pkg.go.dev is new to the list this year and was a top resource for 32% of respondents. Respondents who use pkg.go.dev are more likely to agree they are able to quickly find Go packages / libraries they need: 91% for pkg.go.dev users vs. 82% for everyone else.

<img src="survey2020/resources.svg" alt="Bar chart of top 5 resources respondents use to answer Go-related questions" width="700"/>

Over the years, the proportion of respondents
who do not attend any Go-related events has been trending up. Due to Covid-19, this year we modified our question around Go events, and found over a quarter of respondents have spent more time in online Go channels than in prior years, and 14% attended a virtual Go meetup, twice as many as last year. 64% of those who attended a virtual event said this was their first virtual event.

<img src="survey2020/events.svg" alt="Bar chart of respondents participation in online channels and events" width="700"/>

<p id="TOC_12.1">We found 12% of respondents identify with a traditionally underrepresented group (e.g.,
ethnicity, gender identity, et al.), the same as 2019, and 2% identify as women, fewer than in 2019 (3%).
Respondents who identified with underrepresented groups showed higher rates of disagreement with the statement
"I feel welcome in the Go community" (10% vs. 4%) than those who do not identify with an underrepresented group. These questions allow us to measure diversity in the community and highlight opportunities for outreach and growth.</p>

<img src="survey2020/underrep.svg" alt="Bar chart of underrepresented groups" width="700"/>
<img src="survey2020/underrep_groups_women.svg" alt="Bar chart of those who identify as women" width="700"/>
<img src="survey2020/welcome_underrep.svg" alt="Bar chart of welcomeness of underrepresented groups" width="700"/>

We added an additional question this year on assistive technology usage, and found that 8% of respondents are using some form of assistive technology. The most commonly used assistive tech was contrast or color settings (2%). This is a great reminder that we have users with accessibility needs and helps drive some of our design decisions on websites managed by the Go team.

<img src="survey2020/at.svg" alt="Bar chart of assistive technology usage" width="700"/>

The Go team values diversity and inclusion, not simply as the right thing to do, but because diverse voices can illuminate our blindspots and ultimately benefit all users. The way we ask about sensitive information, including gender and traditionally underrepresented groups, has changed according to data privacy regulations and we hope to make these questions, particularly around gender diversity, more inclusive in the future.


## Conclusion {#conclusion}

Thank you for joining us in reviewing the results of our 2020 developer survey!
Understanding developers’ experiences and challenges helps us measure our progress and directs the future of Go.
Thanks again to everyone who contributed to this survey—we couldn't have done it without you. We hope to see you next year!




# Contact information

- Your Name: Genki Sugimoto
- Email Address: cfhoyuk.reccos.nelg@gmail.com
- GitHub Username: Genki-S
- Twitter Username: @GenkiSugimoto
- IM Address: None
- IRC Nickname: None
- Webpage/Blog: http://genkisugimoto.com
- College/University: Waseda University
- Subject/Major: Computer Science, Software Engineering

# Project

## Project Name

PickyTest (, PickeySpec, DiffTest, DiffSpec, or much cooler names I get along the project)

## Project Description

Run tests you should run, omit tests which is not related to your changes.

This project tries to determine which test should be run for a given commit based on changes introduced since the base commit (likely to be the latest commit on master branch). It maintains the data of relationships between application code and test cases, for example,  "line 10 of `a.rb` is executed by test cases A, B, and C". In this way, when a commit which changes line 10 of `a.rb` is introduced, we know we should run test cases A, B, and C (and more, based on other changes). Using this, we can skip test cases which is not influenced by changes introduced.

We call this relationships "code-to-test mappings" later in this document.

The idea will be mostly the same as [Predicting Test Failures | Tenderlovemaking](http://tenderlovemaking.com/2015/02/13/predicting-test-failues.html). Only my project will be available for users with much easier way (by installing a gem and writing some setup code).

## Why did you choose this idea?

1) This has been my area of interest

I am majoring in software engineering and the productivity in software engineering has been my interest for a long time. My dissertation is named "Improving Fault Localization Based on Dynamic Slicing using Additional Assertions", in which I used dynamic slicing (which can determine which lines of code had influence on certain line) to detect buggy code programmatically. I don't know why, but I'm really enthusiastic about improving productivity on software engineering (which you can confirm on my outstanding 1700+ commits [dotfiles repository](https://github.com/Genki-S/dotfiles)).

2) My experience at Cookpad made me realize test-execution time is the most potent problem

I had been working at Cookpad Inc. from May 2014 to January 2015, during which I realized the more significant problem in software engineering lies in test-execution time rather than in bug-detecting time. From then on, I investigated and tried a lot of approaches like [nviennot/rspec-console](https://github.com/nviennot/rspec-console) (for local development) and [cookpad/rrrspec](https://github.com/cookpad/rrrspec) (for CI).

3) This has been the idea of my master thesis

One day, I realized that I can be selective about which test cases to run for a commit if I could know which test cases were influenced by the changes introduced. However, I have been overwhelmed by the variety of testing frameworks and the variety of open source projects I want to use as research subjects. So, I felt this is a good opportunity to focus on Minitest (and RSpec, because I use it) as testing frameworks and Rails (which, by the way, I really love) as a subject for my research.

## Please describe an outline project architecture or an approach to it

This project requires 2 components: one is an application (which we call "pickytest-gem") which generates code-to-test mappings and determines which test cases should be run for a given commit, and the other is a permanent storage (which we call "pickytest-storage") which stores the code-to-test mappings of the latest stable commit (because generating code-to-test mappings requires running all test cases, we cannot perform it for every test run in order to achieve the performance improvement we are seeking).

### pickytest-gem

The responsibilities of this gem are as follows:

1. Generate code-to-test mappings for a given commit and register it to pickytest-storage (this will be a "base" commit to calculate test cases to run for a given commit)
2. Determine which test cases should be run for a given commit, using the base commit registered to pickytest-storage

Both can be run on any CIs. Users will be able to use it by only writing some setup code in, say, `spec_helper.rb` file. Running only selected test cases on CIs might be a bit tricky, so I might implement a feature to generate another directory named `selected_tests` or `selected_specs` which contains files with selected test cases. In that case, users can run all selected test cases easily.

### pickytest-storage

The responsibilities of this storage are as follows:

1. Store "base" code-to-test mappings for repositories
2. Provide APIs to register and get those "base" lines-to-test mappings

This should be a simple storage. I am planning to make it a web application because everyone wants to run their tests on CIs. In that case, if there are no storage on the web, users should do one of the following:

- Commit file(s) which contains code-to-test mappings to VCS
- Prepare storage to store and get code-to-test mappings by themselves

Which is not smart, so I am planning to provide a simple web storage along with pickytest-gem.

## Give us details about the milestones for this project

- May 25: Project Start
- June 1: pickytest-gem prototype (generate code-to-test mappings, select test cases given code-to-test mappings and a commit) is ready
- June 8: pickytest-storage prototype (store code-to-test mappings for repositories) is ready
- June 15: pickytest ecosystem (pickytest-gem and pickytest-storage working seemlessly) is ready
- June 16: Beta release
- June 16-30: Gather feedbacks and improve on them
- July 1-14: Plan an experiment which can estimate how effective this approach is
- July 15: Release
- July 15-: Maintenance and data gathering
- August 9: Publish quick analysis of gathered data

## Why will your proposal benefit Ruby on Rails?

Because test execution time on Travis CI is really long! And this approach is expected to cut down the test execution time by quite a significant amount. This will benefit Ruby on Rails because people can quickly know if a pull request breaks something, which makes development faster.

# Open Source

## Please describe any previous Open Source development experience

Recently I became able to submit pull requests to open source projects. My enthusiasm lies in Vim, so my contributions are mostly for Vim related projects. I submitted a [pull request to vimperator/vimperator-labs](https://github.com/vimperator/vimperator-labs/pull/157) and some vim plugins like [saihoooooooo/glowshi-ft.vim](https://github.com/saihoooooooo/glowshi-ft.vim/pull/8), [takac/vim-hardtime](https://github.com/takac/vim-hardtime/pull/12) and [Shougo/unite.vim](https://github.com/Shougo/unite.vim/pull/516).

## Why are you interested in Open Source?

Because it's open!

Here are big reasons why I love open source:

- I can learn a ton
  - by reading source code
  - by interacting with other developers around the world
- I can have big dreams
  - of creating popular project which can gain 10000+ stars
  - of becoming a committer of popular projects
- I can be confident to use external open source libraries
  - because I myself can fix it, at least
- I can enjoy writing code!
  - emoji, LGTM, etc...

# Availability

## How long will the project take? When can you begin?

I prospect this project can be released in 8 weeks. I want to conduct an experiment using this, and it will take some more weeks. Of course the maintenance continues forever.

I can start as soon as "Student coding" starts, which means May 25, 2015.

## How much time do you expect to dedicate to this project? (weekly)

I can use most of my time for this project if I were approved as GSoC participant. I want to say 24/7, but to be realistic, I say I can devote 8 hours a day, 5 days a week.

## Where will you be based during the summer?

I am planning to stay in Silicon Valley this summer.

## What timezone will you be working in and what hours do you plan to work? (so we can best match mentors)

I will be working in GMT -7:00 timezone. I plan to work from 07:00 to 17:00 (I am a super early bird).

## Do you have any commitments for the summer? (holidays/work/summer courses)

I have a workshop from August 13 to August 27, and I will not be available during the period. Other than that, I have no commitments.

# Other

## Have you ever participated in a previous GSoC? If yes, describe your project.

No.

## Have you applied for any other 2015 Summer of Code projects? If yes, which ones?

No.

## Why did you apply for the Google Summer of Code ?

The biggest reason is that I think this is a great opportunity to jump into OSS community. I have sent pull requests to some small projects and I have been dreaming of becoming a committer of big projects. However, participating in communities of those big projects is scary for me because I think my skill is not enough to participate in those big projects. Using GSoC, I can interact with people in OSS community and I might be able to get precious feedbacks, with which I can know what skills I need in order to join OSS community more actively.

So, in short, I want to be confident to join OSS community through GSoC.

## Why did you choose Ruby on Rails as a mentoring organization?

- Because I used Ruby on Rails intensively with my part-time job at Cookpad Inc.
- Because I love Ruby on Rails and the community around it (there are many useful gems out there), and I want to participate in that community
- Because the idea of ["Improve Rails Testing Ecosystem"](https://github.com/railsgsoc/ideas/wiki/2015-Ideas#improve-rails-testing-ecosystem) matches my area of enthusiasm

## Why do you want to participate and why should Ruby on Rails choose you?

I want to participate because I want an opportunity to start committing to Ruby on Rails and other projects around it. Ruby on Rails wants to choose me because I will be a contributor to Ruby on Rails community after GSoC ends. As I noted previously, I will use GSoC to know what I need to learn in order to join OSS community. Because Ruby on Rails is the framework I use the most and I love the most, I want to contribute to it in the future.

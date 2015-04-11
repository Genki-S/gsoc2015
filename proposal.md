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

This project consists of one gem (which we call "pickytest").

1. Generate code-to-test mappings for a given commit
2. Determine which test cases should be run for a given commit using (1) the code-to-test mappings for a "base" commit (likely to be the merge-base of the given commit and the HEAD of master branch) and (2) VCS diff information between the given commit and the "base" commit.

Both can be run on any CIs. Users will be able to use it by only writing some setup code in, say, `spec_helper.rb` file. Running only selected test cases on CIs might be a bit tricky, so I might implement a feature to generate another directory named `selected_tests` or `selected_specs` which contains files with selected test cases. In that case, users can run all selected test cases easily.

### Approaches

#### 1. Use Ruby 2.3.0-dev+ to use `Coverage.peek_result`

This approach takes the same approach described in this article: [Predicting Test Failures | Tenderlovemaking](http://tenderlovemaking.com/2015/02/13/predicting-test-failues.html).

The overall idea is to peek in coverage information after each test run and calculate code-to-test mappings. The problem of this approach is that this needs `Coverage.peek_result` method, which is not present in current stable releases of Rubies. I noticed Ruby 2.3.0-dev has this method, so this approach forces users to either (1) patch Ruby themselves with the patch which add `Coverage.peek_result` method or (2) use Ruby 2.3.0-dev. In my opinion, this is not desirable comparing to the second approach.

#### 2. Run each test case from scratch (requiring all files for every run)

The first approach needed `Coverage.peek_result` because of these 2 limitations:

- To get coverage, Ruby source files should be required or loaded between `Coverage.start` and `Coverage.result`
- `Coverage.result` clears out all the coverage information

In code, the limitations looks like this:

[a_method.rb](https://gist.github.com/Genki-S/f698a688ca4f6ef7c74b) (gist link)

```
def a_method
  s = 0
  10.times do |x|
    s += x
  end
  s
end
```

[experiment.rb](https://gist.github.com/Genki-S/cef9766b6ead493a2218) (gist link)

```
require 'coverage'

Coverage.start
require './a_method.rb'  # => true
a_method
p Coverage.result        # => {"/Users/.../a_method.rb"=>[1, 1, 1, 10, nil, 1, nil]}

# Does not produce coverage because `Coverage.result` clears out the coverage
# p Coverage.result      # coverage measurement is not enabled (RuntimeError)

# Does not produce coverage because the file `a_method.rb` is not loaded
Coverage.start
a_method
p Coverage.result        # => {"/Users/.../a_method.rb"=>[]}

# Does not produce coverage because the file `a_method.rb` is not loaded
Coverage.start
require './a_method.rb'  # => false
a_method
p Coverage.result        # => {"/Users/.../a_method.rb"=>[]}

# Unload `a_method.rb`
$".delete_if { |s| s.include?('a_method.rb') }

# Does produce coverage because the file `a_method.rb` is loaded
Coverage.start
require './a_method.rb'  # => true
a_method
p Coverage.result        # => {"/Users/.../a_method.rb"=>[1, 1, 1, 10, nil, 1, nil]}
```

This means we can generate code-to-test mappings in this procedure:

1. start new Ruby process or unload all loaded files
2. call `Coverage.start`
3. `require` all files
4. run single test case
5. call `Coverage.result` and save code-to-test mappings
6. loop 1 to 5 for all test cases
7. combine all code-to-test mappings into one file

This is expected to be extremely slow. But I think this is a bearable overhead to reduce test execution time later.

For quick experiment, I executed a single test in Rails repository and benchmarked it. The result was like this:

(machine spec)
```
MacBook Pro (Retina, 13-inch, Late 2013)
Processor 2.8 GHz Intel Core i7
Memory 16 GB 1600 MHz DDR3
```

(benchmark)
```
$ n=0; time ( while (( n++ < 10000 )); do ruby -w -Itest test/mail_layout_test.rb -n test_explicit_class_layout; done )
119.15s user 24.78s system 97% cpu 2:27.68 total
```

119.15s for 100 execution, so it took roughly 1.2s for this one test case to run.

Let's say test preparation (like loading files) costs 1.2 seconds and one test case run costs C seconds on average in Rails repository. Because Rails has about 11000 test cases if I get it right (`$ grep -R 'def test_' . | wc -l` resulted in 10758), it will take at least 3.5 hours ((11000 * 1.2).to_f / 60 / 60 = 3.67) for only test preparations. Total time will hugely depend on the constant C, but I believe it will not exceed 24 hours.

## Give us details about the milestones for this project

- May 25: Project Start
- June 1: Decide which approach to take
- June 8: pickytest prototype (generate code-to-test mappings, select test cases given code-to-test mappings and a commit) is ready
- June 9-15: Try using it in some repositories and improve
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

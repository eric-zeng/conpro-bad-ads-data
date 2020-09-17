## Overview

This repository contains a labeled dataset of low quality, misleading,
"clickbait", or otherwise potentially problematic online advertisements, found
on the most popular news websites and misinformation websites on the internet.
We describe findings from this dataset in our paper, [Bad News: Clickbait and
Deceptive Ads on News and Misinformation Websites](https://homes.cs.washington.edu/~ericzeng/ConPro_Ads.pdf).

### What's in the dataset
- Screenshots of online ads and their landing pages
- Metadata for each ad:
  - URL of the site the ad was found on (parent site)
  - URL of the ad's landing page
  - Whether the parent site was categorized as mainstream news or misinformation
  - Platform/library used by the website to run the ad
  - Whether the ad was categorized as a native ad or display ad

### IMPORTANT
The screenshots folder is too large for this git repository! Please download
the tar.gz archive from [homes.cs.washington.edu/~ericzeng/conpro_screenshots.tar.gz](homes.cs.washington.edu/~ericzeng/conpro_screenshots.tar.gz)
and uncompress it into a folder called `/screenshots`.

### Files
- `index.html`: A static HTML UI for viewing the dataset, which makes it easier
  to explore the dataset.
- `dataset.csv`: The primary table that contains our coded dataset. Each row in
  this table represents a single ad, and contains the label applied to the ad,
  the filename of the screenshot of the ad, and metadata about the ad, the page
  it appeared on, and the landing page of the ad.
- `screenshots/`: This directory contains screenshots of ads and landing pages
  in our directory. For metadata on these screenshots, refer to `dataset.csv`.


## Methodology
For a full description of our data collection and labeling methodology, please
refer to our [paper](https://homes.cs.washington.edu/~ericzeng/ConPro_Ads.pdf).

### Input Datasets
We collected a list of 6714 mainstream news sites using the Alexa Web Information
Service, including all sites under the "News" category and subcategories that
include "News and Media" or "Magazines and E-Zines".

We collected a list of 1158 known misinformation sites, compiled from a combination
of existing sources, including Politifact, Snopes, Media Bias/Fact Check, OpenSources,
PropOrNot, FakeNewsCodex, and the "Alternative News" category in the Alexa Web Information Service.

### Crawling the Ads
We crawled ads using a web crawler built with Puppeteer, a Chromium-based browser
automation framework. We crawled the home page of each site in
our input dataset, as well as an article from each site (found by extracting RSS feeds). For each page we crawled, our crawler used CSS selectors from the Easylist blocklist to identify ads on web pages, it took individual screenshots of each ad, and it clicked the ad to take a screenshot of the landing page. We also stored additional metadata, like the HTML subtree of each ad. We crawled each domain using a clean browser profile with no history, and isolated each crawler instance in a Docker container to ensure that no traces of previous crawls remained in the file system. We performed the crawls for this dataset between January 15-19, 2020.

### Labeling the Ads
We generated a qualtitative codebook to describe different types of problematic ads we observed in a preliminary analysis of the dataset, with each code describing a set of ads with similar advertisers, products, and advertising tactics. Our codes ranged from ads for things that could cause material harm, such as potentially misleading ads for supplements and investment pitches, to ads that people find irritating, such as ads for celebrity news content farms. The codebook was informed by prior academic research, regulations, and journalism on deceptive advertising, clickbait, malvertising, and advertising industry practices, such as
- https://www.ftc.gov/reports/blurred-lines-exploration-consumers-advertising-recognition-contexts-search-engines-native
- https://www.nytimes.com/2016/10/31/business/media/publishers-rethink-outbrain-taboola-ads.html
- https://www.usenix.org/conference/usenixsecurity16/technical-sessions/presentation/nelms

Of the 55,045 ads that we crawled, we manually labeled a subset of 5414 ads,
choosing ads that appeared on the top 100 ranked misinformation and news sites,
as well as a set of 100 news sites with similar popularity rankings to the set
of misinformation sites (to allow for comparisons controlled for popularity).
After excluding ads that were occluded by other page elements, uninitialized,
or did not have a screenshot due to measurement error, we analyzed a total of
2421 ads with content. Labeling was performed by one researcher with expert
domain knowledge.

The codes used to label the ads are as follows:

#### Problematic Ads
- *Content Farms:* News sites and blogs that contain a high density of ads, often broken up into slideshows to artificially increase ads loaded. The content of the articles are typically about human interest news, celebrity news, or political news.
- *Insurance Advertorials:* Ads appearing to be news articles about people saving money on car or health insurance, to persuade consumers to give personal information to insurance companies for quotes. The landing page does not clearly disclose that it is an ad.
- *Mortgage Advertorials:* Ads for mortgage refinancing, promising large savings, sometimes citing changes to government policies. The goal is to collect consumers' personal information and send it to lenders for quotes. Unclear advertising disclosure.
- *Investment Pitches:* Ads for investment opportunities that make sensationalist claims about their returns, ``secret stock picks'', or predictions of imminent economic turmoil. The advertisers are not affiliated with established brokerages or financial institutions.
- *Misleading Political Polls:* Ads that appear to be political opinion polls, about politically polarizing candidates or issues, but require users to submit names and email addresses \dash likely for fundraising or advertising purposes.
- *Potentially Unwanted Software:* Ads for software downloads that primarily consist of misleading UI elements, like large buttons labeled "Download" or "Watch Now", rather than advertising the name of the product or its functionality.
- *Product Advertorials:* Ads for consumer products written in the style of a blog post or news article that do not obviously disclose that they were written by the advertiser, other than in the fine print in the header or footer of the page.
- *Sponsored Editorial:* Articles hosted on news sites paid for and/or authored by an advertiser, to sell products or promote their views.
- *Sponsored Search:* Ads for products or travel packages, but rather than linking to a specific business, links to search results for the product.
- *Supplements:* Ads for supplements which claim about solve various chronic medical conditions, such as tinnitus, dark spots, weight loss, and toe nail fungus, but are not FDA approved.

#### Benign Ads
- *Charities / PSAs:* Charitable causes, public service announcements, class action lawsuit settlements, and other ads in the public interest.
- *Political Campaigns:* Ads for political candidates or advocacy organizations, intended to spur people into taking action, including voting, signing petitions, donating, or other forms of political participation.
- *Products and Services:* Straightforwards ads for various consumer products. No deception about the intent or identity of the ad is used.
- *Self Links:* Ads that link to a page on the parent domain. Some native ad platforms will recommend both sponsored content and 1st party articles from the publisher.

### Ad Platforms and Libraries
We also labeled the ad platforms or publisher side libraries for many of the
ads in the dataset, using two approaches. First, we detected well-known platforms
like Google Ad Manager, Taboola, and Outbrain using CSS selectors that match HTML classes that we determined to be associated with the platform, based on manual inspection. Second, for each DOM subtree we detected as an ad, we recorded each third-party resource in the subtree (iframes, anchors, images, and scripts), as well as any modifications made to the subtree via third-party Javascript elsewhere in the document. Post-crawl, we manually identified the publisher-side ad platform or other entity (e.g., ad exchange or third-party image host) behind the 100 most popular third-party resources \dash we did this by examining the resources and reading promotional materials or documentation at the domain of the resources. Lastly, we labeled the ad platforms we identified in both approaches as either native ad platforms or display ad platforms, based on how they describe their own product on their websites.

## Notes and Errata
- Some entries in the dataset include no screenshot of the ad, or no screenshot
  of the landing page, because of issues with specific ads that did not
  render properly, or ads that only opened landing pages when specific elements
  were clicked, or other measurement errors. We still labeled ads with missing screenshots if enough information was available to make a decision.

## About
This dataset was produced by Eric Zeng, Tadayoshi Kohno, and Franziska Roesner
at the Paul G. Allen School of Computer Science and Engineering at the University of Washington.

If you use this dataset in your research, please cite the following paper:

[Eric Zeng, Tadayoshi Kohno, and Franziska Roesner, "Bad News: Clickbait and
Deceptive Ads on News and Misinformation Websites" in Workshop on Technology and
Consumer Protection (ConPro ’20), 2020.](https://homes.cs.washington.edu/~ericzeng/ConPro_Ads.pdf)

If you have questions, feel free to email: [ericzeng@cs.washington.edu](mailto:ericzeng@cs.washington.edu).

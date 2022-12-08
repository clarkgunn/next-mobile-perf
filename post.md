# Mobile Performance of Next.js Sites

## Background

Next.js is one of the most popular React frameworks and is seeing heavy adoption.
I am currently working on performance remediation for a large e-commerce site built with Next. While the site has numerous 3rd party analytics, observability, and clientside A/B testing scripts, the performance bottlenecks I am facing are mostly due to large app, vendor, and framework JavaScript bundles.

![Next.js logo](./next-logo.png)

## Mobile Web Performance

While modern phones often sport impressive specs similar to desktop ones, due to thermal constraints, network volatility, and the additional necessary background work, only a portion of that device power translates to web performance. Progressive Performance (Chrome Dev Summit 2016) is a great talk that goes into the constraints of mobile devices and their impact on performance. [Youtube link](https://www.youtube.com/watch?v=4bZvq3nodf4)

For my project, mobile performance is my largest concern, desktop scores are mostly fine. 75% of site traffic, however, comes from mobile, and the site has failing web vital scores for many page types on mobile. I wanted to get a view of where my project sits in the performance landscape of mobile sites built with Next.js so I sought some data to compare.

## Methodology

Next.js includes a [Showcase page](https://nextjs.org/showcase) for sites built with the framework. The list contains many enterprise, Fortune 500 companies and submissions can be proposed via a github discussion thread.

I scraped the links and then verified Next.js was used on the linked page. I threw out `giveindia.org` which redirected to a site not built with Next. `Jet.com` seems to have been acquired by Walmart since it now redirects there, fortunately, `walmart.com` is using Next so I just swapped Jet for Walmart.

With this list, I generated a list of links pointing to the [PageSpeed Insights](https://pagespeed.web.dev) scores of each Next URL. I manually copied the Web Vital and Lighthouse scores to a spreadsheet. (I could have automated this but was leery of introducing bugs that might impact scores)

PageSpeed Insights includes web vital scores from the Chrome User Experience Report (CrUX) for the last 28 days. It also provides a Lighthouse audit run with preconfigured specs and a Pass/Fail assessment score. A Passing score is given if the three Core Web Vital scores, Largest Contentful Paint (LCP), First Input Delay (FID), and Cumulative Layout Shift (CLS) are green for the last 28-day period.

In my spreadsheet, I included: The Pass/Fail assessment, the three Core Web Vital scores, other web vitals: First Contentful Paint (FCP), Interaction to Next Paint (INP), and Time To First Byte (TTFB), and just the main Lighthouse performance score.

## Results

The spreadsheet is available at [this repo](https://github.com/ClarkMitchell/next-mobile-perf). The scores in the spreadsheet were fetched on December 5, 2022. The links in the URL column should open the relevant PageSpeed Insights page and may show varying scores from what's recorded here.

![Image of a 110 row spreadsheet of the mobile web vitals scores of sites built with the Next.js framework. Spreadsheet available at https://github.com/ClarkMitchell/next-mobile-perf](./next-showcase-mobile-web-vitals.png)

## Interpretation

Out of 110 sites:

✅ 27 are Passing with all green CWV.

❌ 80 are failing with 1 or more CWV scores.

⏸️ 3 Had insufficient data and were not included in CrUX

These results are not a glowing endorsement of Next performance-wise. Given the work I've been doing for the past 8 months, ~73% of Next sites failing CWV on mobile isn't surprising to me now. But for me from 2 years ago, still in the honeymoon phase with Next.js, this was a bit of a kick in the gut. I believed that Next would provide a "pit of success", and that most of my performance concerns would involve React Memo. I don't think I was alone in this belief.

There are many instances of the Lighthouse score being at odds with the CrUX data, highlighting that we shouldn't over-rely on any one type of data. Staples has perhaps made some very recent performance improvements, which would be one explanation for why the Lighthouse score is at such odds with the trailing CrUX scores.

PageSpeed Insights uses MOTO G4, which is considered a good low-end device for performance testing, for the Lighthouse audit. Next.js sites perform abysmally with this device emulation with 87 / 110 getting "poor" results. As device access increases around the world, median specs are decreasing rather than increasing, so performance on devices with specs like the Moto G4 should not be thrown out as an outlier.

The spreadsheet includes a Median and Average row, but to get a more intuitive overview, I grouped the Good, Needs Improvement, and Poor scores for each metric and graphed them as a stacked bar chart.

### Overview Table

|            | Good | Needs Improvement | Poor |
| ---------- | ---- | ----------------- | ---- |
| LCP        | 41   | 42                | 24   |
| FID        | 89   | 9                 | 6    |
| CLS        | 77   | 18                | 12   |
| FCP        | 43   | 48                | 16   |
| INP        | 13   | 43                | 48   |
| TTFB       | 38   | 55                | 14   |
| Lighthouse | 2    | 18                | 87   |

![stacked bar chart of Good, Needs Improvement, and Poor scores for each Web vital plus Lighthouse. Available in the overview sheet of the spreadsheet](./overview-chart.png)

FID and INP scores are the opposite of what I would have intuited. FID is mostly a load time score, and I expect sites with large Javascript bundles and long hydration tasks to have poor FID. INP is still an experimental metric but is meant to cover all interactions and not just the first. I would expect SPA-like client-side interactivity to do better with INP after hydration but the opposite is true for Next, good FID, and poor INP.

LCP ideally is independent of any accompanying JavaScript framework, so long as LCP images are shipped in the initial HTML and/or preloaded, but out of all the CWV scores this was the one Next.js sites struggled with the most. An interesting follow-up inquiry might be what percent of Next sites serve LCP image sources in the server-rendered HTML and what percent use the `next/image` package.

In my case, where constraints prevent server rendering and preloading of LCP images, the performance bottleneck is the framework and app bundles. It is a bit unfair of me to say "I need the framework to show my images" and also "The framework is preventing my images from loading quickly", but finding ways to break up Next app bundle sizes has led to the most LCP gains in my work.

All of these sites have different requirements in terms of interactivity and 3rd party and are likely hosted on a variety of platforms. Any given performance issue for any individual site can't be attributed to Next with this data alone, but the one thing these sites have in common is Next.js, and the overall picture is negative.

## Http Archive

We can compare Next to other technologies using a report available from Http Archive.

![Line chart of percent of sites achieving good core web vital scores using all technologies in the Http Archive data set](./cwv.png)

According to [this report](https://datastudio.google.com/reporting/55bc8fad-44c2-4280-aa0b-5f3f0cd3d2be/page/M6ZPC?params=%7B%22df44%22:%22include%25EE%2580%25800%25EE%2580%2580IN%25EE%2580%2580ALL%22%7D) only about 40% of sites are getting a good score for all three Core Web Vitals. So thats not great for mobile users of the web, which is to say all of us, but if we add Next to this report we get an even starker picture.

![Line chart comparing Next Core Web Vital Scores against an average line of all other technologies from Http Archive.](./next-vs-all.png)

[Report link](https://datastudio.google.com/reporting/55bc8fad-44c2-4280-aa0b-5f3f0cd3d2be/page/M6ZPC?params=%7B%22df44%22:%22include%25EE%2580%25800%25EE%2580%2580IN%25EE%2580%2580ALL%25EE%2580%2580Next.js%22%7D) in the Http Archive data set.

Next performs worse on mobile than the average site. 25.9% of Next sites have good CWV scores in October and this percentage is close to the 24.5% of passing sites from the Showcase data.

## Going Forward

Next 13 is introducing a new architecture with React Server Components meant to decrease the amount of JavaScript sent to the client, but server components require logic to parse the transfer protocol, and my limited testing with unstable versions has not revealed strong performance gains.

Additionally, the changes required to achieve these benefits would nearly constitute a rewrite for my current project. Overall I'm not bullish on Next.js for projects where performance is a vital consideration.

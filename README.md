# Tweetoscope

Real-time trending-hashtag pipeline over a stream of tweets. A Java monolith 
(single-threaded, ~2-4k tweets/min ceiling, no fault tolerance) was re-architected 
into Kafka-streamed microservices, containerised with Docker.

**Full code, commit history, and live CI/CD pipeline: [GitLab repo](<lien>)**

Built at CentraleSupélec (software-engineering course project) with Emmanuel Benichou 
and Margaux Blondel.

## Architecture

Five decoupled services communicating through Kafka topics (`rawtweets → tweetsfiltered 
→ hashtagextracted → hashtagcounts`), partitioned for parallel throughput and replicated 
(factor 3, 3 brokers) for fault tolerance:

| Service | Role |
|---|---|
| **Producer** | Publishes the tweet stream (mocked, since the X/Twitter API is now paid) |
| **Filter** | Filters tweets (e.g. by language) via Kafka Streams |
| **Hashtag Extractor** | Extracts hashtags from filtered tweets |
| **Hashtag Counter** | Maintains a live leaderboard of hashtag counts |
| **Visualizer** | Renders the top-5 hashtags as a live histogram |

## Stack

Java 21 · Kafka · Docker · Kubernetes · Maven · GitLab CI/CD · JaCoCo · SpotBugs · Checkstyle

## CI/CD

A 5-stage GitLab pipeline runs on every commit: build (5 service JARs) → quality_check 
(Checkstyle, SpotBugs) → test (JUnit) → coverage (JaCoCo) → deploy (auto-published report 
and PDF to GitLab Pages).

## Kubernetes

Zookeeper runs as a multi-node ensemble to avoid a single point of failure, and 
Kubernetes auto-restarts failed pods. The backend chain (Producer → Filter → Extractor 
→ Counter) deploys and streams correctly. The Visualizer pod currently fails 
(`java.awt.AWTError`: no X11 display in the headless cluster), so full end-to-end 
deployment isn't functional yet; the Visualizer runs standalone (Docker/local) 
in the meantime.

## Report

Full write-up: [`Report.pdf`](Report.pdf)
# NVMe VPS: what $2.50 to $96 a month actually buys, how to verify the disk you paid for, and which Vultr plan fits your workload (full pricing table)

NVMe VPS plans start at $2.48 a month, and $4 buys 1 GB of RAM from one provider and 2 GB from another. On Vultr, the cheapest machine with an NVMe disk is **$6/mo**; everything below it runs on regular SSD, including the $2.50 and $5 plans most people find first. What follows is the current Vultr lineup with prices and hourly rates, a benchmark of what NVMe VPS costs elsewhere so you can tell whether a quote is normal, and the two commands that tell you whether the disk you were sold is actually NVMe.

> **Quick pick.** For a database, a CI runner or a container host with real write traffic, most people end up on 2 vCPU / 4 GB with 80–100 GB of NVMe. On Vultr that's $24/mo ($0.036/hr) for 2 vCPU, 4 GB RAM and 100 GB NVMe. If you're still evaluating, take the $6/mo 1 vCPU / 1 GB plan by the hour and destroy it when you're done.
> 👉 [See the current Vultr NVMe plan prices](https://bit.ly/VuLtr)

## Which Vultr plans actually have NVMe

Vultr splits Cloud Compute into two lines, and only one of them is NVMe. High Performance runs on current-generation AMD EPYC with NVMe SSD attached. Regular Performance runs on previous-generation Intel with regular SSD. If NVMe is the reason you're shopping, the Regular Performance table is just there so you can see what you're giving up.

**High Performance (AMD EPYC, NVMe SSD)**

| Plan (vCPU · RAM) | NVMe | Transfer | Price | Deploy |
| --- | --- | --- | --- | --- |
| 1 vCPU · 1 GB | 25 GB | 2 TB | $6.00/mo · $0.009/hr | [Deploy](https://bit.ly/VuLtr) |
| 1 vCPU · 2 GB | 50 GB | 3 TB | $12.00/mo · $0.018/hr | [Deploy](https://bit.ly/VuLtr) |
| 2 vCPU · 2 GB | 60 GB | 4 TB | $18.00/mo · $0.027/hr | [Deploy](https://bit.ly/VuLtr) |
| 2 vCPU · 4 GB | 100 GB | 5 TB | $24.00/mo · $0.036/hr | [Deploy](https://bit.ly/VuLtr) |
| 4 vCPU · 8 GB | 180 GB | 6 TB | $48.00/mo · $0.071/hr | [Deploy](https://bit.ly/VuLtr) |

**Regular Performance (previous-gen Intel, regular SSD, not NVMe)**

| Plan (vCPU · RAM) | SSD | Transfer | Price | Deploy |
| --- | --- | --- | --- | --- |
| 1 vCPU · 0.5 GB (IPv6 only) | 10 GB | 0.5 TB | $2.50/mo · $0.004/hr | [Deploy](https://bit.ly/VuLtr) |
| 1 vCPU · 0.5 GB | 10 GB | 0.5 TB | $3.50/mo · $0.005/hr | [Deploy](https://bit.ly/VuLtr) |
| 1 vCPU · 1 GB | 25 GB | 1 TB | $5.00/mo · $0.007/hr | [Deploy](https://bit.ly/VuLtr) |
| 1 vCPU · 2 GB | 55 GB | 2 TB | $10.00/mo · $0.015/hr | [Deploy](https://bit.ly/VuLtr) |
| 2 vCPU · 2 GB | 65 GB | 3 TB | $15.00/mo · $0.022/hr | [Deploy](https://bit.ly/VuLtr) |
| 2 vCPU · 4 GB | 80 GB | 3 TB | $20.00/mo · $0.030/hr | [Deploy](https://bit.ly/VuLtr) |
| 4 vCPU · 8 GB | 160 GB | 4 TB | $40.00/mo · $0.060/hr | [Deploy](https://bit.ly/VuLtr) |
| 6 vCPU · 16 GB | 320 GB | 5 TB | $80.00/mo · $0.119/hr | [Deploy](https://bit.ly/VuLtr) |
| 8 vCPU · 32 GB | 640 GB | 6 TB | $160.00/mo · $0.238/hr | [Deploy](https://bit.ly/VuLtr) |
| 16 vCPU · 64 GB | 1280 GB | 10 TB | $320.00/mo · $0.476/hr | [Deploy](https://bit.ly/VuLtr) |
| 24 vCPU · 96 GB | 1600 GB | 15 TB | $640.00/mo · $0.952/hr | [Deploy](https://bit.ly/VuLtr) |

Two things to read out of the High Performance table. Moving from 1 vCPU / 2 GB to 2 vCPU / 2 GB costs $6 a month, which is what a second core is worth here; doubling RAM from 2 GB to 4 GB doubles the price. The cheapest plan also has the tightest disk-to-RAM ratio, 25 GB per GB of RAM, while the 4 vCPU / 8 GB plan runs 22.5 GB per GB. Vultr is consistent at roughly 20–25 GB of storage per GB of RAM. A database with a large working set can run out of disk before it runs out of memory.

Vultr also sells High Frequency instances, which use NVMe as well and cost more per vCPU for higher clock speeds. The two tables above are where the price-per-spec decision happens.

## The 7,000 MB/s on the sales page is an interface number

DartNode advertises up to 7,000 MB/s sequential read, 5,000 MB/s write and 1M+ IOPS on NVMe Gen4 x4. Cloudzy publishes a sample run on its NVMe page using `fio --rw=randread --bs=4k --iodepth=64`, which returns 180k IOPS and 702 MiB/s. Both numbers are real and they describe different things: one is the ceiling of the drive's PCIe lane, the other is what a 4K random-read test at queue depth 64 returned on one machine.

Arct Cloud prints the honest version of this on its own NVMe page. Its comparison table lists NVMe PCIe at multiple GB/s, SATA III at around 550 MB/s and a spinning disk at around 160 MB/s, followed by a disclaimer that these figures compare storage interfaces and are not VPS performance guarantees. The same page says there is no single speed multiplier, because VPS storage performance depends on the hardware, the allocation and the workload.

DartNode's own numbers show the label hides a 2× spread: 7,000 MB/s for Gen4, 3,500 MB/s for Gen3, both sold as "NVMe". The drive model, how many tenants share it, and the IOPS ceiling your provider set on the virtual disk matter more than the bus.

You can settle it in about ninety seconds on the machine itself. Run:


lsblk -d -o NAME,ROTA,TRAN


If the transport column says `nvme`, you got NVMe. If it says `sata`, you didn't, whatever the order page said. Cloudzy's page points out the common version of this problem: providers advertise "SSD" and run SATA-attached SSDs underneath it, which is a different bus and a different protocol. Then measure:


fio --rw=randread --bs=4k --runtime=10 --name=t --filename=/tmp/t --size=1G --iodepth=64


Compare the IOPS you get against the plan you paid for, not against the marketing page. A small instance on a busy host will land well below the headline, and that is the number your queries will actually see.

## What an NVMe VPS costs elsewhere

Five providers, all with NVMe across their plan range, priced as of their current pages. Cloudzy's figures are a 50% discount with the list price shown next to it, which makes it the clearest sample here for spotting a first-term price.

| Provider | Entry NVMe plan | Price | Mid-range plan | Price |
| --- | --- | --- | --- | --- |
| Cloudzy | 1 vCPU · 0.5 GB · 20 GB | $2.48/mo (list $4.95) | 2 vCPU · 4 GB · 120 GB | $14.48/mo (list $28.95) |
| Arct Cloud | 1 vCPU · 1 GB · 15 GB | $3.89/mo | up to 300 GB NVMe on top tiers | — |
| OrangeVPS | 1 vCPU · 2 GB · 25 GB | $4/mo | 4 vCPU · 8 GB · 80 GB | $12/mo |
| DartNode | 1 vCPU · 1 GB · 30 GB | $8.95/mo | 4 vCPU · 4 GB · 100 GB | $26.95/mo |
| AsiaWebHost | 2 vCPU · 4 GB · 60 GB | S$55/mo (SGD) | 4 vCPU · 8 GB · 120 GB | S$70/mo (SGD) |
| Vultr (High Performance) | 1 vCPU · 1 GB · 25 GB | $6.00/mo | 2 vCPU · 4 GB · 100 GB | $24.00/mo |

The $4 tier is where the differences are sharpest. Cloudzy gives 1 GB of RAM for $3.48, Arct gives 1 GB and only 15 GB of disk for $3.89, OrangeVPS gives 2 GB for $4. Same money, twice the RAM. At the $8–12 tier it gets worse: DartNode charges $8.95 for 1 vCPU and 1 GB, while OrangeVPS charges $12 for 4 vCPU, 8 GB and 80 GB. Four times the cores for three dollars more.

Disk-to-RAM ratio swings just as far. Cloudzy's 4 GB plan carries 120 GB (30 GB per GB of RAM), DartNode's 8 GB plan carries 200 GB (25×), AsiaWebHost's 16 GB plan carries 240 GB (15×), OrangeVPS's 16 GB plan carries 160 GB (10×). For a database or a logging stack, that ratio tells you how much index and log data fits before you need a resize. For a web server holding 5 GB of files, it tells you nothing.

Vultr sits in the middle on price and is not the cheapest per gigabyte in this group. What you're paying the difference for is hourly billing with a monthly cap of 672 hours, so a machine that runs for six hours costs six hours' worth instead of a month.

## Vultr's $300 credit: what it covers and what it doesn't

New accounts can get **$300 in promotional credit**, applied through the signup link or the promo code field under Billing. A payment method has to be attached, either a card or PayPal, and duplicate accounts aren't eligible. The credit is a balance that gets drawn down as you deploy, not a refund and not something you can withdraw. Unused credit expires after 30 days, and Vultr's refund policy doesn't cover services or credits.

The 30-day window is the constraint that matters, not the dollar amount. A 4 vCPU / 8 GB High Performance instance bills $48 a month; thirty days on it consumes about $48 of the $300. Even the 8 vCPU / 32 GB Regular Performance plan at $160/mo leaves more than $130 unspent when the credit expires. Only the 16 vCPU / 64 GB and 24 vCPU / 96 GB plans get close to burning through it. So plan the trial around what you can finish in a month, and use the credit to test the hardware you'd actually buy: a staging database on the 2 vCPU / 4 GB plan, a load test on the 4 vCPU / 8 GB, a latency check from two or three regions.

One billing detail that catches people: a stopped server still bills. Charges start at deploy and continue while the instance is powered off. Destroy it to stop the meter, and destroy everything before day 30 if you don't intend to pay.

👉 [Start a Vultr account with the $300 credit](https://bit.ly/VuLtr)

## Who should not buy NVMe, and which plans are overkill

Arct Cloud's own FAQ says CPU-bound applications may benefit more from a faster processor tier than from faster storage. That applies here. If your workload is compiling code, running a game server's physics, or transcoding video, the disk is idle most of the time and a High Frequency instance at a lower storage tier will beat a High Performance one at the same price. The workloads that actually feel NVMe are the ones doing frequent small reads and writes: B-tree updates and WAL writes in PostgreSQL or MySQL, container layer extraction during builds, Elasticsearch and Meilisearch index queries, and log ingestion at peak.

Cold storage doesn't need it either. Backups, media archives and infrequently read datasets sit fine on cheap SATA or object storage, and paying NVMe prices for a disk that gets read twice a week is money thrown away.

Two Vultr plans most readers should skip: the $320/mo 16 vCPU / 64 GB and the $640/mo 24 vCPU / 96 GB. They're Regular Performance, so no NVMe, and they exist for people who need a lot of RAM in one box, not for people who need I/O. If you're shopping for NVMe and looking at those, you probably want a High Performance plan two or three sizes down plus a separate storage volume.

## FAQ

**Is an NVMe VPS really faster than SATA?**
For random I/O, yes, by a wide margin. Cloudzy states random I/O on NVMe runs roughly an order of magnitude faster than SATA SSD and roughly two orders of magnitude faster than a spinning disk, and Arct's interface table puts SATA III at around 550 MB/s against multiple GB/s for PCIe. Sequential throughput is the smaller part of the gain; the difference shows up in latency on small, scattered reads, which is what databases do.

**Are cheap NVMe VPS plans fake?**
Some "SSD" plans are SATA drives behind a PCIe-sounding label, which is a documentation problem as much as a hardware one. The price itself isn't the tell: Cloudzy sells 20 GB of NVMe for $2.48 and OrangeVPS sells 25 GB for $4. Check the transport with `lsblk` after you deploy, on the first machine, every time.

**How do I verify the disk before I commit to a year?**
Buy hourly or monthly first. Deploy, run `lsblk -d -o NAME,ROTA,TRAN` and the `fio` command above, and compare against what the plan promised. Vultr's hourly billing with a 672-hour monthly cap means a week of testing costs a fraction of the monthly price.

**Should I pay monthly or prepay?**
Prepaying is usually cheaper, and some providers only discount on multi-month terms. The tradeoff is that a prepaid term locks the price and the provider. Given how much the per-dollar specs vary between providers at the $4–12 tier, run one month first and prepay only after the disk checks out.

## How to choose, plan by plan

A small site or a single app that doesn't hammer the disk: the $5 Regular Performance plan with 1 vCPU, 1 GB and 25 GB of SSD is enough, and NVMe buys you nothing there.

A PostgreSQL or MySQL instance with a working set under 50 GB: Vultr's High Performance 2 vCPU / 4 GB with 100 GB NVMe at $24/mo. The 25:1 disk-to-RAM ratio leaves room for the WAL and a couple of indexes, and $0.036/hr keeps a resize cheap if you got it wrong.

Continuous integration, container builds or anything that unpacks layers all day: the 4 vCPU / 8 GB with 180 GB NVMe at $48/mo. Builds are CPU-hungry and I/O-hungry at the same time, and the 4 vCPU plan is the first one where both stop being the bottleneck.

Logging and analytics with heavy sustained writes: start on 2 vCPU / 4 GB and watch the disk, not the CPU. If the ratio is the problem rather than throughput, a plan with more storage per GB of RAM will serve you better than a faster one.

Before you order, confirm two things: that the plan you clicked is in the High Performance line and not Regular Performance, and that it includes an IPv4 address, since the $2.50 entry plan is IPv6-only. Then destroy anything you deploy during the credit window before day 30.

👉 [Pick a Vultr NVMe plan and deploy](https://bit.ly/VuLtr)

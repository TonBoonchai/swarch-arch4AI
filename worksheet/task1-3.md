# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** บุญชัย จินาพร

**Student ID / รหัสนิสิต:** 6632110421

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 489 | 595 | 93.5% | 100% |
| t0-small | 15 | 37 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> The large model buys **accuracy**: 93.5% vs 72.5% for the small model (+21 points), and only the large model meets QR3 (≥ 90%). It costs **latency** and **money**: p50 is 489 ms vs 15 ms (about 32× slower), and in `t0-large` all 200 reviews were paid large-model calls, while `t0-small` made none.

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 28.4 | 225 | 347 | 20 | 0 | 92.8% |
| t1-after | 9.1 | 394 | 1009 | 0 | 24 | 92.6% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> p50 is calculated only from requests that got an answer. The 20 requests that hung are left out, even though each of those users waited 10 s and got nothing, so p50 (225 ms) looks better than the baseline (489 ms). The real story is in `errors` (20 of 200 failed) and the run time (28.4 s vs about 10 s for a healthy run).

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> A hung call doesn't just slow down one review. While it waits, the client, the gateway and the review service each hold a connection (and a slot) open for it. With only 10 requests in flight, each hung request blocks a slot for 10 s, time in which about 20 normal reviews could have gone through. So everyone queues behind it: the whole run took 28.4 s instead of about 10 s.

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> Pending reviews should not stay unlabelled. They should be labelled **later, in the background**: a batch job regularly picks up the stored pending reviews (the review service already lists them at `GET /reviews/pending`), sends them to the model and saves their labels, while no user is waiting. This is **T4, asynchronous / batch**.

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> At **0.1 s**, the timeout is shorter than a normal large-model answer (p50 489 ms in `t0-large`), so almost every review times out and is saved as `pending` with a null label. Users get fast answers, but almost no labels (this breaks FR2). At **10 s**, a hung call still holds its slot for 10 s, so the system behaves like `t1-before`. This breaks QR2 (at most 1 s), and the client also gives up at 10 s, so users still see errors instead of a pending review.

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 7 | 540 | 91.5% | 38% |
| t2-0.7 | 0.7 | 364 | 589 | 93.0% | 53% |
| t2-0.8 | 0.8 | 486 | 694 | 93.5% | 64% |
| t2-0.9 | 0.9 | 466 | 583 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> I would deploy threshold **0.7**. Every threshold from 0.6 up meets QR3 (≥ 90%) and QR1 (p95 < 1 s), and 0.6 uses the fewest large calls (38%), but its 91.5% accuracy is only 1.5 points above the limit. With 0.7 the accuracy is 93.0% (a 3-point margin) and 53% of reviews go to the large model. I give up some **cost** (15 points more large calls than 0.6) to keep an **accuracy margin**. Going to 0.8 would add 11 more points of large calls for only 0.5 points more accuracy.

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> The cascade only speeds up the reviews the small model answers alone. Reviews sent to the large model are as slow as before. p95 sits at the slow end, which is always made of large-model reviews, so it barely changes (589 ms vs 595 ms in `t0-large`). p50 only drops a lot when more than half of the reviews are small only: at 0.6 (62% small only) p50 is 7 ms, but at my threshold of 0.7 only 47% are small only, so the middle review went to the large model and p50 is 364 ms.

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> The trust is misplaced when the small model is **confidently wrong**, for example with negations, because it reads words one by one. In the test data, *"Honestly the salad is not cold."* is a positive review, but the small model labels it negative with confidence 0.702. That's ≥ my threshold of 0.7, so the large model, which gets it right, is never asked. The restaurant owner then sees more negative reviews than are real, thinks customers are unhappier than they are, and might change things that were not a problem at all.

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 6 | 580 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> Short, common phrases benefit, because many customers write exactly the same text (for example, "cold food" appears 9 times and "delicious!" 8 times in the 200 reviews). In `t3-cache`, 35% of requests were cache hits, and large calls dropped from 53% to 38%. Long, detailed reviews never benefit: their wording is very specific, so even one different word gives a different cache key, and a text that appears only once (94 of the 200) can never be a hit.

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> On a cache hit the model isn't called, so after the deploy the cache would keep returning the **old** model's labels. The most common reviews would never get the better model's answers, and old and new labels would be mixed. On deploy day the cache must be **cleared**, so every review goes to the new model once and the cache refills with new answers. To make this safe by design, I would **mark each entry with the model version** (for example, put it in the key: `"large-v2|cold food"`), so an old model's answer can never be returned for the new model.

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> I expected the cache to help most requests (around 75%), but in `t3-cache` only 35% were cache hits. A hit needs exactly the same text, and only the short, common phrases repeat (125 of the 200 texts were different). I learned that how much a cache helps depends on how often users send exactly the same input, and you only find that out by measuring.

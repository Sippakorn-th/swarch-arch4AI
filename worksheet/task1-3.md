# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** Sippakorn Thunyahan

**Student ID / รหัสนิสิต:** 6631355721

This is **individual work**: each student completes and submits their own copy, in their own fork.

_งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง_

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run      | p50 ms | p95 ms | accuracy | large calls |
| -------- | ------ | ------ | -------- | ----------- |
| t0-large | 498    | 596    | 93.5%    | 100%        |
| t0-small | 9      | 21     | 72.5%    | 0%          |

**Q0.** What does the large model buy, and what does it cost?

large model: better accuracy, less throughput (can handle less request within the same time period compared to smalle model)
_large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร_

>

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run       | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
| --------- | ------------ | ------ | ------ | ------ | ------- | -------- |
| t1-before | 26.5         | 247    | 426    | 20     | 0       | 92.8%    |
| t1-after  | 9.7          | 401    | 1010   | 0      | 28      | 93.0%    |

**Q1.1** In `t1-before`, p50 can look _better_ than the baseline. Why is p50 misleading here? Which column tells the real story?

The load test calculates p50 only from successful requests, error tells the real story

_ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง_

>

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.
Even though async allow other requests run the waiting request still occupies resources, for example, an open connection between services and request state in memory
_การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway_

>

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

A review can be saved as pending without a positive ornegative label. Then Later, we can propose background prediction and retries using T4/asynchronous processing.

_รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น_

>

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

if 0.1, there could be request that actually dont take much time but will be aborted, while 10, its too long to catch the request that take too much time

_เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที_

>

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run    | threshold | p50 ms | p95 ms | accuracy | large calls |
| ------ | --------- | ------ | ------ | -------- | ----------- |
| t2-0.6 | 0.6       | 6      | 535    | 91.5%    | 38%         |
| t2-0.7 | 0.7       | 361    | 577    | 93.0%    | 53%         |
| t2-0.8 | 0.8       | 450    | 567    | 93.5%    | 64%         |
| t2-0.9 | 0.9       | 477    | 588    | 93.5%    | 77%         |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

0.8 over 0.7 for slightly higher accuracy, we give up time

>

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?
when the slower path dominate, the at95 point change drastically
_ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก_

>

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

at the end, its a model estimate, it can be confidently wrong, or uncertain about right answer, for example curse word can be view as negative, but ;fucking good; is actually positive, so user could get the wrong idea about the comment if not review manually
_cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้_

>

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run      | p50 ms | p95 ms | accuracy | large calls | cache hits |
| -------- | ------ | ------ | -------- | ----------- | ---------- |
| t3-cache | 7      | 551    | 93.5%    | 40%         | 34%        |

**Q3.1** Which reviews benefit from the cache, and which never will?

normalized text matching a previously cached review can reuse its prediction without evaluating the models again. a unique reviews wont benefit

>

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?
reset cache automatically everytime the model change
_สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร_

>

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.
failed requests can increase while the reported median became lower
_ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture_

>

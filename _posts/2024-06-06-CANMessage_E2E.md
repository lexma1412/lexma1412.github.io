---
layout: default
title:  "CAN Message E2E"
---

# CAN Message E2E

* In communication via CAN (also other protocols), the aspect to deal with error of message need to be considered. E2E is one of mechanism to prevent data error and Asymmetric error (Refer [1]).The cause of error is vary from HW Random fault (CAN register/memory is corrupt) or systematic fault (the implementation does not ensure the FFI), and it lead to some fault under consideration:<br />

(1) Loss of communication between ECUs<br />
(2) Unintended repetition of message<br />
(3) Order of data is changed unexpectedly<br />
(4) Bit of data is changed unexpetectedly<br />
(5) Timing is not correct<br />

* (1) (2) (3) (4) (5) can be detected or corrected by checksum and/or alive counter and CRC mechanism. Depend on design, these mechanism can be implemented in Application or BSW layer, with my experience, target BSW will invoke callout function to check or insert alive counter and CRC.

## Checksum


## Alive counter
Every time CAN message is sent,  sender shall increase alive counter by 1, the requirement for alive counter is specified by OEM, usually range of alive counter is 0..14 so it takes 4 bits in payload, alive counter will be reset when it reach the limit. If alive counter of 2 consecutive message is not changed, it can be jugded that message is freezed or repeated unexpectedly or if the increasement of alive counter is not continuous, timing may not be correct.


## CRC
CAN frame has 16 bit for CRC slot, CRC 8bit is usually used to prevent data error.To verify CRC, software or hardware CRC module can be used.
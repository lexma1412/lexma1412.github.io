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

* (1) (2) (3) (4) can be detected or corrected by checksum and/or alive counter and CRC mechanism.

## Checksum
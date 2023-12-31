---
title: '🌐 네트워크 스터디 3주차'
date: 2023-11-22 22:51:00
category: 'Study'
draft: false
---

### 슬라이딩 윈도우가 뭘까요?
슬라이딩 윈도우는 통신 프로토콜에서 사용되는 기술로, 송신자와 수신자 간의 효율적인 데이터 전송을 위해 일정 범위의 패킷을 유지하면서 전송하는 방식입니다.
### Go Back N에 대해 설명해주세요.
Go Back N은 오류 발생 시, 송신측이 마지막으로 성공적으로 수신된 패킷 이후의 모든 패킷을 재전송하는 오류 복구 메커니즘으로, 일부 패킷 손실 시 전체를 재전송합니다.
### Selective Repeat에 대해 설명해주세요.
elective Repeat은 송신자가 일부 패킷만 재전송하는 오류 복구 메커니즘으로, 수신자는 손상된 패킷만을 요청하여 효율적인 데이터 전송을 가능케 합니다.
## 📌 TCP
### TCP에 대해 설명해주세요.
TCP(Transmission Control Protocol)는 신뢰성 있는 데이터 전송을 제공하는 연결 지향형 프로토콜로, 패킷 손실, 중복, 순서 문제 등을 처리하여 안정적인 통신을 보장합니다.
### 3 way handshake에 대해 설명해주세요.
3-way handshake은 TCP 연결 설정을 위한 과정으로, 클라이언트가 서버에게 SYN을 보내고, 서버가 이를 수락하여 SYN-ACK를 보내며, 클라이언트가 다시 ACK를 보내 연결을 설정합니다.
### 4 way handshake에 대해 설명해주세요.
4-way handshake은 TCP 연결 종료를 위한 과정으로, 클라이언트가 FIN을 보내고, 서버가 이를 수락하여 ACK를 보내며, 서버가 FIN을 보내고 클라이언트가 이에 대한 ACK를 보내 연결을 종료합니다.
### TCP 빠른 재전송에 대해서 설명해주세요.
TCP 빠른 재전송은 패킷 손실 감지 시, 별도의 타이머를 기다리지 않고 바로 재전송을 시도하여 빠르게 오류 복구를 시도하는 TCP의 기능입니다.
### Congestion control에 대해 설명해주세요.
Congestion control은 네트워크 혼잡을 피하기 위한 TCP의 기능으로, 네트워크 부하를 감지하고 전송 속도를 동적으로 조절하여 안정적인 통신을 유지합니다.
### Flow control에 대해 설명해주세요.
Flow control은 TCP에서 수신자가 송신자의 전송 속도를 제어하는 메커니즘으로, 수신자가 처리할 수 있는 양의 데이터만을 송신자에게 허용함으로써 네트워크 혼잡을 방지합니다.
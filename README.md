<div align="center">
  <h1>《손으로 배우는 대형 모델》 시리즈 프로그래밍 실습 튜토리얼</h1>

  <a href="https://img.shields.io/badge/version-v0.1.0-blue">
    <img alt="version" src="https://img.shields.io/badge/version-v0.1.0-blue?color=FF8000?color=009922" />
  </a>
  <a>
    <img alt="Status-building" src="https://img.shields.io/badge/Status-building-blue" />
  </a>
  <a>
    <img alt="PRs-Welcome" src="https://img.shields.io/badge/PRs-Welcome-red" />
  </a>
  <a href="https://github.com/gidaseul/dive-into-llms/stargazers">
    <img alt="stars" src="https://img.shields.io/github/stars/gidaseul/dive-into-llms" />
  </a>
  <a href="https://github.com/gidaseul/dive-into-llms/network/members">
    <img alt="FORK" src="https://img.shields.io/github/forks/gidaseul/dive-into-llms?color=FF8000" />
  </a>
  <a href="https://github.com/gidaseul/dive-into-llms/issues">
    <img alt="Issues" src="https://img.shields.io/github/issues/gidaseul/dive-into-llms?color=0088ff"/>
  </a>

  <p>
    <a href="#project-motivation">프로젝트 소개</a> /
    <a href="#tutorial-catalog">튜토리얼 목록</a> /
    <a href="#contributors">기여자</a>
  </p>
</div>

## 💡 업데이트

2025/06/06 기준, 다음 두 가지 업데이트가 반영되었습니다.

- [x] 화웨이 Ascend 커뮤니티와 함께 제작한 공익 튜토리얼 《대형 모델 개발 전 과정》 공개
- [x] 기존 실습 튜토리얼 내용 보강 및 신규 주제 추가: 수학 추론, GUI Agent, 대형 모델 정렬, 스테가노그래피 등

<a id="project-motivation"></a>

## 🎯 프로젝트 소개

《손으로 배우는 대형 모델》 시리즈 프로그래밍 실습 튜토리얼은 상하이교통대학교의
`자연어 처리 프론티어 기술`(NIS8021) 및 `인공지능 보안 기술`(NIS3353) 강의 자료를 확장한 공개 학습 자료입니다.
대형 모델 관련 내용을 입문 수준에서 직접 실습해 볼 수 있도록 구성했으며, 과정 설계와 연구 입문에 바로 활용할 수 있도록 하는 것을 목표로 합니다.

본 저장소는 공익 목적의 무료 자료입니다.

<a id="tutorial-catalog"></a>

## 📚 튜토리얼 목록

| 주제 | 소개 | 자료 |
| --- | --- | --- |
| 미세조정과 배포 | 사전학습 모델을 특정 작업에 맞게 미세조정하고, 추론 및 데모 배포까지 이어지는 흐름을 다룹니다. | [교안](./documents/chapter1/dive-into-llm.pdf) / [문서](./documents/chapter1/README.md) / [노트북](./documents/chapter1/dive-tuning.ipynb) |
| 프롬프트 학습과 사고 연쇄 | 대형 모델 API 호출, 제로샷/퓨샷 프롬프팅, 사고 연쇄 기반 추론 기법을 소개합니다. | [교안](./documents/chapter2/dive-into-prompting.pdf) / [문서](./documents/chapter2/README.md) / [노트북](./documents/chapter2/dive-prompting.ipynb) |
| 지식 편집 | 언어 모델이 보유한 특정 지식을 수정하는 대표적 방법과 검증 절차를 다룹니다. | [교안](./documents/chapter3/dive_edit_0410.pdf) / [문서](./documents/chapter3/README.md) / [노트북](./documents/chapter3/dive_edit.ipynb) |
| 수학 추론 | 수학 데이터 증류와 지도 미세조정을 통해 수학 추론 능력을 학습시키는 흐름을 실습합니다. | [교안](./documents/chapter4/math.pdf) / [문서](./documents/chapter4/README.md) / [노트북](./documents/chapter4/sft_math.ipynb) |
| 모델 워터마킹 | 생성 텍스트에 워터마크를 삽입하고 검출 및 평가하는 방법을 살펴봅니다. | [교안](./documents/chapter5/watermark.pdf) / [문서](./documents/chapter5/README.md) / [노트북](./documents/chapter5/watermark.ipynb) |
| 탈옥 공격 | 대형 모델 보안을 이해하기 위해 대표적 탈옥 공격과 평가 관점을 실습합니다. | [교안](./documents/chapter6/dive-Jailbreak.pdf) / [문서](./documents/chapter6/README.md) / [노트북](./documents/chapter6/dive-jailbreak.ipynb) |
| 대형 모델 스테가노그래피 | 생성 텍스트 속에 은닉 정보를 삽입하는 스테가노그래피 기법을 소개합니다. | [교안](./documents/chapter7/stega.pdf) / [문서](./documents/chapter7/README.md) / [노트북](./documents/chapter7/llm_stega.ipynb) |
| 멀티모달 모델 | 멀티모달 대형 언어 모델의 구조와 학습 및 추론 방식을 정리합니다. | [교안](./documents/chapter8/mllms.pdf) / [문서](./documents/chapter8/README.md) / [노트북](./documents/chapter8/mllms.ipynb) |
| GUI Agent | GUI 환경에서 동작하는 에이전트의 동작 방식과 예제를 소개합니다. | [교안](./documents/chapter9/GUIagent.pdf) / [문서](./documents/chapter9/README.md) / [노트북](./documents/chapter9/GUIagent.ipynb) |
| 에이전트 보안 | 개방형 에이전트 환경에서의 위험 인지와 평가 방법을 다룹니다. | [교안](./documents/chapter10/dive-into-safety.pdf) / [문서](./documents/chapter10/README.md) / [노트북](./documents/chapter10/agent.ipynb) |
| RLHF 안전 정렬 | PPO 기반 RLHF 실험을 통해 보상 모델과 정책 최적화 과정을 실습합니다. | [교안](./documents/chapter11/RLHF.pdf) / [문서](./documents/chapter11/README.md) / [노트북](./documents/chapter11/RLHF.ipynb) |

## 🔥 신규 공개: 《대형 모델 개발 전 과정》

화웨이 Ascend와 함께 제작한 공익 튜토리얼 《대형 모델 개발 전 과정》도 함께 공개되어 있습니다.
기존 실습 시리즈를 바탕으로, Ascend 생태계를 활용한 대형 모델 개발 흐름을 초급부터 고급까지 단계적으로 다룹니다.
PPT, 실험 매뉴얼, 영상 자료를 포함한 형태로 구성되어 있습니다.

- 학습 페이지: [대형 모델 개발 학습 존](https://www.hiascend.com/edu/growth/lm-development#classification-floor-1)

<p align="center">
  <img src="./pics/icon/title.jpg" width="48%"/>
  <img src="./pics/icon/cover.png" width="48%"/>
  <img src="./pics/icon/team.png" width="48%"/>
  <img src="./pics/icon/agent.png" width="48%"/>
</p>

## 🙏 면책 조항

이 튜토리얼의 내용은 기여자들의 개인 경험, 공개 자료, 일상적인 연구 및 실습 축적을 바탕으로 정리되었습니다.
모든 내용은 학습과 참고를 위한 것이며, 완전한 정확성을 보장하지는 않습니다.
문제가 있거나 개선 제안이 있다면 Issue 또는 PR로 알려주세요.

## 🤝 기여 환영

이 저장소는 계속 보완 중인 프로젝트입니다.
오탈자 수정, 내용 개선, 추가 예제, 문서 정리 등을 포함한 모든 기여를 환영합니다.

<a id="contributors"></a>

## ❤️ 기여자

이 프로젝트를 지원하고 기여해 주신 분들께 감사드립니다.

**《손으로 배우는 대형 모델》 시리즈 개발팀**

- 상하이교통대학교: [장줘성](https://bcmi.sjtu.edu.cn/home/zhangzs/), [위안퉁신](https://github.com/Lordog), [마신베이](https://scholar.google.com/citations?user=LpUi3EgAAAAJ&hl=zh-CN&oi=ao), [허즈웨이](https://zwhe99.github.io), [두웨이](https://scholar.google.com/citations?user=tFYUBLkAAAAJ&hl=en), [자오하오둥](https://dongdongzhaoup.github.io/), [우쭝루](https://zrw00.github.io/), [우정](https://wuzheng02.github.io/), [둥링중](https://github.com/LZ-Dong), [장위룽](https://aslan-yulong.github.io/)
- 싱가포르국립대학교: [페이하오](http://haofei.vip/)

**《대형 모델 개발 전 과정》 시리즈 개발팀**

- 상하이교통대학교: [장줘성](https://bcmi.sjtu.edu.cn/home/zhangzs/), [류궁선](https://infosec.sjtu.edu.cn/DirectoryDetail.aspx?id=75), [천싱위](https://scholar.google.com/citations?user=d-dNtjrMJ5YC&hl=en), [청펑저우](https://scholar.google.com/citations?user=qxnwzDUAAAAJ&hl=en), [둥링중](https://github.com/LZ-Dong), [허즈웨이](https://zwhe99.github.io), [쥐톈제](https://scholar.google.com/citations?user=f8PPcnoAAAAJ&hl=en), [마신베이](https://scholar.google.com/citations?user=LpUi3EgAAAAJ&hl=zh-CN&oi=ao), [우정](https://scholar.google.com/citations?hl=zh-CN&user=qBM1UbUAAAAJ&view_op=list_works&gmla=AIfU4H6PG9JyjRub6BYIIZ4isQE7MBAM3Eoec6OJfX4z_8-pOE8bI1Wgdo3XL5qOZWR3U-h-lIP2q0zXt5gzyFKMSg7MNnBBWLv5d1IVG30UANczTP0), [우쭝루](https://zrw00.github.io/), [옌쯔허](https://scholar.google.com/citations?user=O2YfSHoAAAAJ&hl=zh-CN), [야오야오](https://scholar.google.com/citations?user=tLMP3IkAAAAJ), [위안퉁신](https://github.com/Lordog), [자오하오둥](https://dongdongzhaoup.github.io/)
- Huawei Ascend 커뮤니티: ZOMI, 셰첸, 청리밍, 러우리화, 자오쩌위

## 🌟 Star History

[![Star History Chart](https://api.star-history.com/svg?repos=gidaseul/dive-into-llms&type=Date)](https://star-history.com/#gidaseul/dive-into-llms&Date)

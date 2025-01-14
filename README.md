2023 성균관대 하계집중 산학협력프로젝트 VAIV
## GPT 기반의 자연스럽고(Friendly) 윤리적인(Harmless) 일상 대화형 챗봇 모델

#  연구 배경 및 목적
    GPT-NEOX(Polyglot-ko) 기반 자연스럽고 윤리적인 한국어 기반 일상 대화형 챗봇 모델 구현
![image](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/18bb1ab4-8924-4b43-b538-1e6529297217)
  
# 개발 내용
- Self-Instruct: GPT4를 이용한 데이터 증강
- RLHF(Reinforcement Learning from Human Feedback): 사람의 선호도를 반영한 강화학습
- DeepSpeed: 대규모 분산 딥러닝을 위한 새로운 메모리 최적화 기술

    - Task 1: 강화학습 단계별 데이터셋 구축
    - Task 2: SFT 모델 Instruction-tuning
    - Task 3: Reward 모델 ver1,2,3 구현
    - Task 4: RLHF와 DeepSpeedChat을 통한 최종 모델 구현 (https://huggingface.co/Trofish/KULLM-RLHF)

# Task1. 강화학습 단계별 데이터셋 구축
![image](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/4bb56e36-0c49-4d15-a2c6-2824867419a8)
![Screenshot 2024-06-18 at 11 05 55 AM](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/2f637065-fa25-4402-b319-113ff4c6e1a9)
![Screenshot 2024-06-18 at 11 06 08 AM](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/2a6c2e9b-1292-43b9-b5e7-5ced3643988d)



# Task2. SFT 모델 Fine-tuning
## Baseline Model
[- 고려대학교 NLP & AI 연구실과 HIAI 연구소가 개발한 한국어 LLM **"KULLM"** 사용](https://github.com/nlpai-lab/KULLM)

## Datasets
![image](https://github.com/VAIV-2023/VAIV2023/assets/79634774/085610db-3714-43c3-855b-58baad2f4e8b)

## SFT Model Finetuning 
![image](https://github.com/VAIV-2023/VAIV2023/assets/79634774/0f5e36fa-20a8-43f9-bd03-5f8224d5e9d0)
* 모델학습에는 Google Colab에서 제공하는 A100 40GB GPU 사용
  
## SFT Model Evaluation
![image](https://github.com/VAIV-2023/VAIV2023/assets/79634774/9fe9e5aa-6dc7-4c7b-8529-45e0a75db9c6)
![image](https://github.com/VAIV-2023/VAIV2023/assets/79634774/a994a960-db7c-4e75-a11a-d7755d372722)
* G-Eval: https://arxiv.org/abs/2303.16634


# Task3-1. Reward Model ver1 구현
## Baseline Model
- EleutherAI에서 개발한 초거대 한국어 언어 모델 **Polyglot-Ko** 사용
- 1.3b 모델과 5.8b 모델을 각각 실험
## Datasets
![image](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/0082da9b-b0b8-4089-8647-cffa5ce724fb)
- InstructGPT의 데이터셋 구축 방법
    - Reward 모델 학습 데이터셋으로 SFT 학습에 사용한 prompt(1,500개 - 일상대화:혐오표현=2:1)와 새로운 prompt(1,000개 - DeepSpeedChat 번역 데이터셋) 사용 
    - SFT 모델에서 한개의 prompt당 K개의 Response를 생성하고, 순위를 Labeling
- 데이터셋 라벨링
    - Instruct GPT의 경우 사람이 직접 Labeling을 하엿지만, 일관된 평가와 시간 단축을 위해 GPt-4와 G-Eval을 이용
    - SFT에서 생성한 두 Response 중 G-Eval 평가 점수 합이 높은 것을 Chosen response로 결정
    - 데이터셋 유형별로 G-Eval 평가 Prompt에 차이를 두었음
    -   ![image](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/7d7117d0-02e9-42dd-8ce3-5244cf726bf8)
## Reward v1 Model Finetuning
![image](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/da4d9b15-ec91-44bb-84d9-f28aeffd16ad)
- InstructGPT 논문에 따르면, Reward 모델은 overfitting되면 성능이 크게 저하된다고 함 --> epoch 수를 1로 설정
- batch size나 learning rate 등 다른 hyper-parameter는 성능에 큰 영향이 없다고 함
- Colab A100 40GB 기준 총 학습 시간 4분

## Reward v1 Model Evaluation
![image](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/c21be612-b26d-4a1c-a1e2-6a99442660da)

- Reward Model Template
  - "아래는 작업을 설명하는 명령어입니다. 요청을 적절히 완료하는 응답을 작성하세요. \n\n ### 명령어:\n{prompt}\n\n ### 응답:\n"

# Task3-2. Reward Model ver2 구현
## Reward Model ver1 Issues
- 구현된 Reward Model의 성능이 좋지 않음 (Accuracy 0.65)
- Reward Model ver1을 사용하여 Step3 학습시 혐오표현이 아닌데도 혐오표현이라고 인식하고 답변하는 문제 발생

## Issue 해결방안
![image](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/6f4f0665-a8c7-4903-a626-f37018b7e4c9)
- SFT 모델로 답변을 2개 생성하였을 때(Ver1), Chosen, Rejected 답변의 차이가 크게 없어 모델이 학습되지 않는 현상을 방지하기 위하여 2개의 모델 **(ChatGPT, SFT)**를 사용하여 답변을 생성(Ver2)
- General Task 답변에 대한 평가 성능을 높이기 위해 Evol-instruct 데이터 추가
- 학습에 사용한 모든 데이터셋은 15 token 이하, cosine 유사도 0.5 이상일 경우 제거하는 Filtering 작업 수행
- 혐오표현 학습시(Ver1) Step3 강화학습 이후에 답변이 이상하게 생성되는 Issue가 있어, 혐오표현을 데이터를 제거하고 학습(Ver2)
- RM-ver1은 GPT4가 Chosen, Rejected 레이블링을 진행하였지만, Resource 이슈로 인해 일부만 사람이 라벨링 진행
    - 일상대화 데이터셋
        - ChatGPT와 SFT 모두 일관되게 높은 퀄리티의 답변을 생성하지 않아, 사람이 직접 라벨링 진행
    - RLHF 한국어 번역, Evol-Instruct 데이터셋
        - ChatGPT가 일관되게 높은 퀄리티의 답변을 생성하여 ChatGPT를 Chosen, SFT를 Rejected로 라벨링 진행
## Reward Model ver2 Evaluation
![image](https://github.com/VAIV-2023/RLHF-Korean-Friendly-LLM/assets/79634774/834cb645-7909-464b-b072-635aaac8eeff)

# Task4. RLHF와 DeepSpeedChat을 통한 최종 모델 구현
- Microsoft에서 만든 대규모 분산 딥러닝을 위한 새로운 메모리 최적화 기술(DeepSpeed)을 RLHF Process에 적용한 DeepSpeedChat 사용
- Human preference로 학습을 시킨 Reward 모델과 강화학습을 통해 SFT 모델에 사람의 선호도를 반영하여 자연스럽고(FRIENDLY), 윤리적인 (HARMLESS) 챗봇 생성
  
## Baseline Models
- Actor Model: KULLM-SFT-V2
- Reward Model: Polyglot-Ko-Reward-V3

## Training Options
![image](https://github.com/VAIV-2023/VAIV2023/assets/79634774/ae2cdfe5-7552-4009-a99a-244e79d945dc)

## RLHF Training
![image](https://github.com/VAIV-2023/VAIV2023/assets/79634774/3d4dbf68-5222-4f6a-a6d0-87ea176c5211)
- 학습 결과, SFT 모델의 답변에 대한 퀄리티인 Reward가 상승하는 것을 확인 (사람의 선호도가 높은 답변을 생성)

## RLFH Model Evaluation
![image](https://github.com/VAIV-2023/VAIV2023/assets/79634774/2b58ed3a-7ed5-4e60-ba4b-c9b291b1fdff)
![image](https://github.com/VAIV-2023/VAIV2023/assets/79634774/75b2a1ee-d7c0-4ba9-ab2f-727abab644e9)

## Final RLHF Model
- https://huggingface.co/Trofish/KULLM-RLHF


# Contributors 🙌 
- 박성완 (성균관대학교 소프트웨어학과 20학번, waniboyy@gmail.com)
- 송현빈 (성균관대학교 소프트웨어학과 20학번, shbin0519@gmail.com)
- 허유민 (성균관대학교 소프트웨어학과 21학번, ymheo1123@gmail.com)
- 홍여원 (성균관대학교 소프트웨어학과 20학번, ryeowon13@gmail.com)


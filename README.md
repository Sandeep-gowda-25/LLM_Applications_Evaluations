# LLM_Applications_Evaluations
Internal logics from frameworks in calculating evaluation metrics for LLM Applications



### Ragas response relevancy:
#### (takes response generated and user_input)

##### Instruction:
Generate a question for the given answer and Identify if answer is noncommittal. Give noncommittal as 1 if the answer is noncommittal and 0 if the answer is committal. A noncommittal answer is one that is evasive, vague, or ambiguous. For example, "I don't know" or "I'm not sure" are noncommittal answers
##### Fewshot examples:
```
1. response relevance i/p: Albert Einstein was born in Germany.
response relevance o/p: 
question = Where was Albert Einstein born?, 
noncommittal=0
2. I don't know about the  groundbreaking feature of the smartphone invented in 2023 as am unaware of information beyond 2022.
question="What was the groundbreaking feature of the smartphone invented in 2023?",
noncommittal=1
```

return output : embedding on actual question for cosine similarity against embedding on generated questions(mean)return cosine_sim * int(not committal count)

### Ragas Context Recall
#### (user_input, retrieved_contexts, response)

##### Instruction:
 "Given a context, and an answer, analyze each sentence in the answer and classify if the sentence can be attributed to the given context or not. Use only 'Yes' (1) or 'No' (0) as a binary classification. Output json with reason."

##### Oneshot input:
  ```
  question="What can you tell me about albert Albert  Einstein?",

  context=
  "Albert Einstein (14 March 1879 - 18 April 1955) was a German-born theoretical physicist, widely held to be one of the greatest and most influential scientists of all time. Best known for developing the theory of relativity, he also made important contributions to quantum mechanics, and was thus a central figure in the revolutionary reshaping of the scientific understanding of nature that modern physics accomplished in the first decades of the twentieth century. His mass-energy equivalence formula E = mc2, which arises from relativity theory, has been called 'the world's most famous equation'. He received the 1921 Nobel Prize in Physics 'for his services to theoretical physics, and especially for his discovery of the law of the photoelectric effect', a pivotal step in the development of quantum theory. His work is also known for its influence on the philosophy of science. In a 1999 poll of 130 leading physicists worldwide by the British journal Physics World, Einstein was ranked the greatest physicist of all time. His intellectual achievements and originality have made Einstein synonymous with genius.",
  answer="Albert Einstein born in 14 March 1879 was  German-born theoretical physicist, widely held to be one of the greatest and most influential scientists of all time. He received the 1921 Nobel Prize in Physics for his services to theoretical physics. He published 4 papers in 1905.  Einstein moved to Switzerland in 1895",

 Output: 

 ContextRecallClassifications(
                classifications=[
                    ContextRecallClassification(
                        statement="Albert Einstein, born on 14 March 1879, was a German-born theoretical physicist, widely held to be one of the greatest and most influential scientists of all time.",
                        reason="The date of birth of Einstein is mentioned clearly in the context.",
                        attributed=1,
                    ),
                    ContextRecallClassification(
                        statement="He received the 1921 Nobel Prize in Physics for his services to theoretical physics.",
                        reason="The exact sentence is present in the given context.",
                        attributed=1,
                    ),
                    ContextRecallClassification(
                        statement="He published 4 papers in 1905.",
                        reason="There is no mention about papers he wrote in the given context.",
                        attributed=0,
                    ),
                    ContextRecallClassification(
                        statement="Einstein moved to Switzerland in 1895.",
                        reason="There is no supporting evidence for this in the given context.",
                        attributed=0,
                    ),
                ]
            ),
 ```
 recall return value : sum of all classification/attributed values divided by total length of responses

### Ragas context precision
#### user_input, retreived_contexts, response

##### Instruction:
 'Given question, answer and context verify if the context was useful in arriving at the given answer. Give verdict as "1" if useful and "0" if not with json output.'

##### fewshot examples:

 ```examples = [
        (
            QAC(
                question="What can you tell me about Albert Einstein?",
                context="Albert Einstein (14 March 1879 – 18 April 1955) was a German-born theoretical physicist, widely held to be one of the greatest and most influential scientists of all time. Best known for developing the theory of relativity, he also made important contributions to quantum mechanics, and was thus a central figure in the revolutionary reshaping of the scientific understanding of nature that modern physics accomplished in the first decades of the twentieth century. His mass–energy equivalence formula E = mc2, which arises from relativity theory, has been called 'the world's most famous equation'. He received the 1921 Nobel Prize in Physics 'for his services to theoretical physics, and especially for his discovery of the law of the photoelectric effect', a pivotal step in the development of quantum theory. His work is also known for its influence on the philosophy of science. In a 1999 poll of 130 leading physicists worldwide by the British journal Physics World, Einstein was ranked the greatest physicist of all time. His intellectual achievements and originality have made Einstein synonymous with genius.",
                answer="Albert Einstein, born on 14 March 1879, was a German-born theoretical physicist, widely held to be one of the greatest and most influential scientists of all time. He received the 1921 Nobel Prize in Physics for his services to theoretical physics.",
            ),
            Verification(
                reason="The provided context was indeed useful in arriving at the given answer. The context includes key information about Albert Einstein's life and contributions, which are reflected in the answer.",
                verdict=1,
            ),
        ),
        (
            QAC(
                question="who won 2020 icc world cup?",
                context="The 2022 ICC Men's T20 World Cup, held from October 16 to November 13, 2022, in Australia, was the eighth edition of the tournament. Originally scheduled for 2020, it was postponed due to the COVID-19 pandemic. England emerged victorious, defeating Pakistan by five wickets in the final to clinch their second ICC Men's T20 World Cup title.",
                answer="England",
            ),
            Verification(
                reason="the context was useful in clarifying the situation regarding the 2020 ICC World Cup and indicating that England was the winner of the tournament that was intended to be held in 2020 but actually took place in 2022.",
                verdict=1,
            ),
        ),
        (
            QAC(
                question="What is the tallest mountain in the world?",
                context="The Andes is the longest continental mountain range in the world, located in South America. It stretches across seven countries and features many of the highest peaks in the Western Hemisphere. The range is known for its diverse ecosystems, including the high-altitude Andean Plateau and the Amazon rainforest.",
                answer="Mount Everest.",
            ),
            Verification(
                reason="the provided context discusses the Andes mountain range, which, while impressive, does not include Mount Everest or directly relate to the question about the world's tallest mountain.",
                verdict=0,
            ),
        ),
    ]
 ```
 return output:
 denominator = sum(verdict_list) + 1e-10
 numerator = sum([(sum(verdict_list[: i + 1]) / (i + 1)) * verdict_list[i]for i in range(len(verdict_list))])

 score = numerator / denominator

### Ragas Faithfullness 
#### user_input, response

#### Instruction1 - Response to mutliple statements
 "Given a question, an answer, and sentences from the answer analyze the complexity of each sentence given under 'sentences' and break down each sentence into one or more fully understandable statements while also ensuring no pronouns are used in each statement. Format the outputs in JSON."

#### oneshot example: statements generation
```examples = [
        (
            StatementGeneratorInput(
                question="Who was Albert Einstein and what is he best known for?",
                answer="He was a German-born theoretical physicist, widely acknowledged to be one of the greatest and most influential physicists of all time. He was best known for developing the theory of relativity, he also made important contributions to the development of the theory of quantum mechanics.",
            ),
            StatementGeneratorOutput(
                statements=[
                    "Albert Einstein was a German-born theoretical physicist.",
                    "Albert Einstein is recognized as one of the greatest and most influential physicists of all time.",
                    "Albert Einstein was best known for developing the theory of relativity.",
                    "Albert Einstein also made important contributions to the development of the theory of quantum mechanics.",
                ]
            ),
        )
    ]
```

#### Instruction2 - faithfullness verdicts with reasoning for each statement
 "Your task is to judge the faithfulness of a series of statements based on a given context. For each statement you must return verdict as 1 if the statement can be directly inferred based on the context or 0 if the statement can not be directly inferred based on the context."

#### Fewshot examples - verdicts generation
```
examples = [
        (
            NLIStatementInput(
                context="""John is a student at XYZ University. He is pursuing a degree in Computer Science. He is enrolled in several courses this semester, including Data Structures, Algorithms, and Database Management. John is a diligent student and spends a significant amount of time studying and completing assignments. He often stays late in the library to work on his projects.""",
                statements=[
                    "John is majoring in Biology.",
                    "John is taking a course on Artificial Intelligence.",
                    "John is a dedicated student.",
                    "John has a part-time job.",
                ],
            ),
            NLIStatementOutput(
                statements=[
                    StatementFaithfulnessAnswer(
                        statement="John is majoring in Biology.",
                        reason="John's major is explicitly mentioned as Computer Science. There is no information suggesting he is majoring in Biology.",
                        verdict=0,
                    ),
                    StatementFaithfulnessAnswer(
                        statement="John is taking a course on Artificial Intelligence.",
                        reason="The context mentions the courses John is currently enrolled in, and Artificial Intelligence is not mentioned. Therefore, it cannot be deduced that John is taking a course on AI.",
                        verdict=0,
                    ),
                    StatementFaithfulnessAnswer(
                        statement="John is a dedicated student.",
                        reason="The context states that he spends a significant amount of time studying and completing assignments. Additionally, it mentions that he often stays late in the library to work on his projects, which implies dedication.",
                        verdict=1,
                    ),
                    StatementFaithfulnessAnswer(
                        statement="John has a part-time job.",
                        reason="There is no information given in the context about John having a part-time job.",
                        verdict=0,
                    ),
                ]
            ),
        ),
        (
            NLIStatementInput(
                context="Photosynthesis is a process used by plants, algae, and certain bacteria to convert light energy into chemical energy.",
                statements=[
                    "Albert Einstein was a genius.",
                ],
            ),
            NLIStatementOutput(
                statements=[
                    StatementFaithfulnessAnswer(
                        statement="Albert Einstein was a genius.",
                        reason="The context and statement are unrelated",
                        verdict=0,
                    )
                ]
            ),
        ),
    ]
```

return output: 
 ratio of total faithfull verdicts(1) to total verdicts

### Ragas Factual Correctness
#### Takes context and response

#### Instruction1: Claim decomposition(it happens on response based on decomposition type)
    "Decompose and break down each of the input sentences into one or more standalone statements. Each statement should be a standalone claim that can be independently verified.
    Follow the level of atomicity and coverage as shown in the examples."
#### Few shot example:
 Input : "Charles Babbage was a French mathematician, philosopher, and food critic."
 Low Automicity and Low Coverage Output : claims=["Charles Babbage was a mathematician and philosopher."]
 Low Automicity and High Coverage Output : claims=["Charles Babbage was a French mathematician, philosopher, and food critic."]
 High Automicity and Low Coverage Output : [
                    "Charles Babbage was a mathematician.",
                    "Charles Babbage was a philosopher.",
                ]
 High Automicty and High Coverage Output : claims=[
                    "Charles Babbage was a mathematician.",
                    "Charles Babbage was a philosopher.",
                    "Charles Babbage was a food critic.",
                    "Charles Babbage was French.",
                ]

#### Instruction2: Natural language inference(reused from faithfullness - applies on context and above statements)
    "Your task is to judge the faithfulness of a series of statements based on a given context. For each statement you must return verdict as 1 if the statement can be directly inferred based on the context or 0 if the statement can not be directly inferred based on the context."

return output calculation:
```tp = sum of verdicts true
fp = sum of verdicts false
if mode is precision, fn = 0
else, fn = sum of verdicts false

if precision, return tp/(tp+fp+1e-8)
if recall, return tp/(tp+fn+1e-8)
```

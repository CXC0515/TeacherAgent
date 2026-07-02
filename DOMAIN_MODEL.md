# 教师 AI 助手领域模型

版本：v0.1  
日期：2026-07-03  
状态：待评审  
适用范围：产品总蓝图阶段

## 1. 建模目标

本文档定义系统底层领域对象、字段、字段属性、关系、证据链和分析结果。

核心目标不是先定义页面，而是先定义：

```text
系统需要理解哪些对象
对象有哪些字段
字段的来源和可信度如何记录
对象之间如何形成多维关系
AI 在哪些环节参与
哪些结果必须由教师确认
```

产品底层逻辑：

```text
教学对象 -> 学生作答证据 -> OCR/多模态识别 -> AI 初评 -> 教师确认 -> 学习诊断 -> 教学行动
```

## 2. 建模原则

### 2.1 对象优先

先定义真实业务对象，再定义页面和接口。

页面只是对象的展示和操作入口，不应反过来决定数据结构。

### 2.2 证据优先

所有重要判断都必须能追溯到证据。

示例：

```text
学生“信息提取能力薄弱”
不能只来自 AI 总结
必须能追溯到：
某次作业 -> 某道题 -> 学生原始图片 -> OCR 文本 -> AI 批改 -> 教师确认
```

### 2.3 关系对象化

关系不能只是外键。

系统中的很多关系有权重、置信度、时间范围和证据，因此关系本身也要建模。

### 2.4 AI 结果可修正

AI 输出必须记录：

```text
来源
置信度
版本
提示词版本
模型版本
教师是否确认
```

### 2.5 语文学科特殊性

语文不能只用“知识点掌握率”建模。

语文至少要同时建模：

```text
文体
题型
能力点
评分点
表达质量
错误类型
文本证据
```

## 3. 通用字段属性

所有核心字段建议采用“值 + 元数据”的结构。

```text
FieldValue
- value: 字段值
- data_type: string / number / boolean / enum / datetime / json / file / relation
- source: manual / ocr / multimodal_ai / llm / system / imported
- confidence: 0.00 - 1.00
- privacy_level: public / internal / sensitive / highly_sensitive
- version: 版本号
- created_at: 创建时间
- updated_at: 更新时间
- created_by: 创建人或系统
- verified_by: 确认人
- verified_at: 确认时间
- verification_status: unverified / confirmed / corrected / rejected
```

说明：

- `manual`：教师或管理员手动录入
- `ocr`：传统 OCR 识别
- `multimodal_ai`：多模态模型识别
- `llm`：大模型推断或生成
- `system`：系统计算得到
- `imported`：从 Excel、学籍系统、题库等导入

## 4. 核心对象总览

### 4.1 人员与组织

```text
Teacher
TeacherRole
Class
Student
StudentGroup
Guardian
StudentRosterImport
```

### 4.2 时间与教学范围

```text
Term
TeachingPeriod
TeachingScope
ScheduleItem
```

### 4.3 教材与课程

```text
Subject
Grade
Textbook
BookVolume
Unit
Lesson
LessonSegment
CurriculumStandard
TeachingObjective
ContentIndex
RetrievalTrigger
```

### 4.4 知识与能力图谱

```text
KnowledgePoint
AbilityPoint
QuestionType
RubricPoint
ErrorType
TextGenre
ExpressionFeature
GraphRelation
```

### 4.5 作业与测评

```text
Assignment
Question
QuestionMaterial
QuestionBankItem
Rubric
RubricCriterion
Submission
AnswerImage
AnswerRegion
RecognizedAnswer
TeacherCorrection
```

### 4.6 批改与诊断

```text
GradingTask
GradingResult
ScoreResult
ErrorFinding
Evidence
AbilityDiagnosis
KnowledgeDiagnosis
ClassDiagnosis
StudentDiagnosis
StudentLongitudinalProfile
ReviewQueue
ReviewPolicy
ReviewConflict
```

### 4.7 教学行动

```text
Intervention
ReviewLessonPlan
RemedialExercise
PersonalizedAdvice
ParentCommunicationDraft
TeachingReflection
```

### 4.8 AI 运行记录

```text
AIJob
AIModel
PromptTemplate
AIOutput
HumanReview
```

## 5. 人员与组织对象

### 5.1 Teacher

教师对象。

```text
Teacher
- id
- name
- phone
- email
- school_name
- default_subject_ids
- default_grade_ids
- created_at
- updated_at
```

隐私说明：

- `phone`、`email` 属于敏感信息。
- MVP 阶段可以不做手机号登录，避免过早引入账号体系。

### 5.2 TeacherRole

教师在特定时间范围内的角色。

```text
TeacherRole
- id
- teacher_id
- role_type: subject_teacher / head_teacher / grade_leader / teaching_research_leader / admin
- class_id
- subject_id
- term_id
- valid_from
- valid_to
```

说明：

同一教师可以同时是语文教师和班主任，因此角色不能直接写死在 Teacher 上。

### 5.3 Class

班级对象。

```text
Class
- id
- name
- grade_id
- school_year
- term_id
- head_teacher_id
- status: active / archived
```

### 5.4 Student

学生对象。

```text
Student
- id
- display_name
- real_name
- student_no
- class_id
- gender
- status: active / transferred / graduated / archived
- privacy_alias
- custom_fields
- deleted_at
```

说明：

- `display_name` 用于界面展示。
- `privacy_alias` 用于脱敏场景，例如“学生 A”。
- 后续如果处理真实学生数据，需要明确数据保存和删除策略。
- `custom_fields` 用于保留未来扩展能力，例如学号规则、分层标签、个性化备注等。

### 5.5 StudentGroup

学生分组。

```text
StudentGroup
- id
- class_id
- name
- group_type: manual / ability_level / temporary / system_generated
- description
```

用途：

- 分层作业
- 小组讲评
- 个别辅导
- 学情对比

### 5.6 StudentRosterImport

学生名单导入记录。

```text
StudentRosterImport
- id
- class_id
- source_file_id
- import_type: excel / csv / manual_batch / external_system
- field_mapping
- total_rows
- success_count
- failed_count
- error_report
- imported_by
- imported_at
```

说明：

- 第一版同时支持手动录入和 Excel 导入。
- 导入后学生信息仍需支持编辑、删除和字段扩展。

## 6. 时间与教学范围对象

### 6.1 Term

学期对象。

```text
Term
- id
- name
- school_year
- semester: first / second
- start_date
- end_date
- status: planning / active / closed
```

### 6.2 TeachingScope

教师当前教学范围。

```text
TeachingScope
- id
- teacher_id
- term_id
- grade_id
- subject_id
- textbook_id
- class_ids
- is_head_teacher
- start_week
- end_week
- progress_status
```

说明：

这是用户提出的“选择当下学期所教授的时间范围、年级教材、是否班主任”的核心对象。

### 6.3 ScheduleItem

教学进度项。

```text
ScheduleItem
- id
- teaching_scope_id
- lesson_id
- planned_week
- planned_date
- actual_date
- status: planned / taught / skipped / delayed
- notes
```

## 7. 教材与课程对象

### 7.1 Subject

学科对象。

```text
Subject
- id
- name
- stage: primary / junior_high / senior_high
```

### 7.2 Grade

年级对象。

```text
Grade
- id
- stage
- name
- order_index
```

### 7.3 Textbook

教材对象。

```text
Textbook
- id
- subject_id
- grade_id
- publisher
- edition
- curriculum_version
- name
```

示例：

```text
初中语文 / 七年级上册 / 统编版
```

### 7.4 BookVolume

册别。

```text
BookVolume
- id
- textbook_id
- name
- volume_order
```

### 7.5 Unit

单元对象。

```text
Unit
- id
- book_volume_id
- name
- unit_order
- theme
- unit_objectives
```

### 7.6 Lesson

课文或课题。

```text
Lesson
- id
- unit_id
- title
- lesson_order
- author
- text_genre_id
- suggested_hours
- source_text
- teaching_focus
```

对于语文：

- `source_text` 可存课文全文或摘要，视版权策略决定。
- 如果不能存全文，应存引用来源、教材页码、结构化摘要。

### 7.7 LessonSegment

课文片段。

```text
LessonSegment
- id
- lesson_id
- segment_type: paragraph / scene / argument / section / poem_line
- order_index
- text
- start_position
- end_position
```

用途：

- 阅读题定位
- 答案证据引用
- 讲评课片段分析

### 7.8 CurriculumStandard

课程标准对象。

```text
CurriculumStandard
- id
- subject_id
- stage
- version
- section_title
- content
- source_reference
```

### 7.9 ContentIndex

教材与资料内容索引。

```text
ContentIndex
- id
- source_object_type: textbook / lesson / curriculum_standard / question_material / teacher_document
- source_object_id
- index_type: keyword / full_text / embedding / hybrid
- index_content
- keywords
- version
- updated_at
```

说明：

- 教材、课文、课程标准可以存全文或结构化内容。
- 检索层应支持关键词、全文和向量混合检索。

### 7.10 RetrievalTrigger

检索触发器。

```text
RetrievalTrigger
- id
- name
- trigger_keywords
- target_index_types
- target_object_types
- retrieval_strategy: keyword_first / semantic_first / hybrid
- min_confidence
- enabled
```

说明：

- 可参考 skill 的触发方式，用少量关键词或意图触发特定资料检索。
- 示例：用户输入“七上散步 作用题”，触发课文、题型、评分点和历史讲评资料检索。

## 8. 知识与能力图谱对象

### 8.1 KnowledgePoint

知识点。

```text
KnowledgePoint
- id
- subject_id
- name
- description
- parent_id
- stage
- grade_range
- tags
```

语文示例：

```text
修辞手法
描写方法
说明方法
文言实词
古诗意象
```

### 8.2 AbilityPoint

能力点。

```text
AbilityPoint
- id
- subject_id
- name
- description
- ability_domain
- parent_id
```

语文能力域示例：

```text
reading_comprehension    阅读理解
information_extraction   信息提取
language_appreciation    语言赏析
structure_analysis       结构分析
theme_understanding      主旨理解
expression_organization  表达组织
critical_thinking        思辨表达
```

### 8.3 TextGenre

文体。

```text
TextGenre
- id
- name
- stage
- description
```

示例：

```text
记叙文
说明文
议论文
文言文
古诗词
非连续性文本
```

### 8.4 QuestionType

题型。

```text
QuestionType
- id
- subject_id
- name
- description
- applicable_genre_ids
```

语文示例：

```text
概括题
含义题
作用题
赏析题
人物形象题
主旨题
文言翻译题
诗歌鉴赏题
```

### 8.5 RubricPoint

评分点。

```text
RubricPoint
- id
- question_id
- description
- expected_keywords
- score
- required: true / false
- scoring_method: exact / semantic / teacher_judged
```

说明：

语文评分点往往不是精确匹配，必须支持语义匹配和教师判断。

### 8.6 ErrorType

错误类型。

```text
ErrorType
- id
- subject_id
- name
- description
- severity
- applicable_question_type_ids
```

语文示例：

```text
漏答要点
答非所问
没有结合文本
表述空泛
术语误用
逻辑混乱
审题偏差
引用错误
表达不完整
```

### 8.7 ExpressionFeature

表达特征。

```text
ExpressionFeature
- id
- name
- description
- polarity: positive / negative / neutral
```

示例：

```text
表达完整
层次清楚
语言空泛
语序混乱
缺少文本依据
```

## 9. 作业与测评对象

### 9.1 Assignment

作业对象。

```text
Assignment
- id
- title
- class_id
- teacher_id
- subject_id
- textbook_id
- unit_id
- lesson_id
- assigned_at
- due_at
- assignment_type: homework / quiz / exam / reading_practice / writing_practice
- status: draft / assigned / collecting / grading / analyzed / archived
```

### 9.2 Question

题目对象。

```text
Question
- id
- assignment_id
- question_no
- title
- material_id
- question_type_id
- full_score
- reference_answer
- rubric_id
- difficulty_level
- source: textbook / workbook / exam_paper / teacher_created / imported
```

### 9.3 QuestionMaterial

题目材料。

```text
QuestionMaterial
- id
- title
- material_type: passage / image / table / poem / classical_text / mixed
- content
- source_reference
- copyright_status: unknown / licensed / public / internal_use
```

语文阅读题通常由“阅读材料 + 多个题目”组成，因此材料要独立建模。

### 9.4 QuestionBankItem

电子题目对象。

```text
QuestionBankItem
- id
- subject_id
- grade_id
- textbook_id
- unit_id
- lesson_id
- source_type: image_recognition / electronic_import / manual / third_party
- source_file_id
- material_id
- question_text
- reference_answer
- rubric_draft
- question_type_id
- knowledge_point_ids
- ability_point_ids
- status: draft / reviewed / active / archived
- created_by
```

说明：

- 第一版应支持题目图片识别和电子版本导入。
- 识别后的题目应进入电子题库，而不是只作为一次性文本使用。
- 题目、参考答案、评分标准都需要支持教师后续编辑。

### 9.5 Rubric

评分规则。

```text
Rubric
- id
- question_id
- total_score
- scoring_description
- version
- created_by
```

### 9.6 RubricCriterion

评分细则。

```text
RubricCriterion
- id
- rubric_id
- criterion_order
- description
- score
- matching_type: keyword / semantic / structure / expression / manual
- required_evidence
```

### 9.7 Submission

学生提交。

```text
Submission
- id
- assignment_id
- student_id
- submitted_at
- submit_source: photo_upload / scan / text_input / file_import
- status: uploaded / recognized / corrected / graded / confirmed
```

### 9.8 AnswerImage

作业图片。

```text
AnswerImage
- id
- submission_id
- file_path
- file_hash
- page_no
- image_width
- image_height
- uploaded_at
- quality_score
- privacy_level
```

### 9.9 AnswerRegion

作答区域。

```text
AnswerRegion
- id
- answer_image_id
- student_id
- question_id
- region_type: full_page / question_block / answer_block / margin_note
- bounding_box
- crop_file_path
- detection_source: layout_model / multimodal_ai / teacher_marked
- confidence
- sequence_order
```

说明：

- 作业图片优先支持整页上传。
- 系统先识别页面边缘和版面，再裁剪出学生作答区域。
- 同一学生的多页作业需要合并到一次 Submission 下。
- 题目识别应在整页结构化之后进行，而不是要求教师一开始手动裁题。

### 9.10 RecognizedAnswer

识别后的学生答案。

```text
RecognizedAnswer
- id
- submission_id
- question_id
- raw_text
- normalized_text
- source: ocr / multimodal_ai / teacher_corrected
- confidence
- image_region
- recognition_model
- recognition_version
- verified_by_teacher
```

### 9.11 TeacherCorrection

教师校对记录。

```text
TeacherCorrection
- id
- recognized_answer_id
- before_text
- after_text
- corrected_by
- corrected_at
- correction_type: typo / segmentation / question_match / student_match / content
```

用途：

- 反映 OCR 质量
- 训练后续识别流程
- 保留原始识别与教师校对之间的证据链

## 10. 批改与诊断对象

### 10.1 GradingTask

批改任务。

```text
GradingTask
- id
- assignment_id
- question_id
- student_id
- recognized_answer_id
- status: pending / ai_graded / teacher_reviewed / confirmed / rejected
- created_at
```

### 10.2 GradingResult

批改结果。

```text
GradingResult
- id
- grading_task_id
- suggested_score
- final_score
- max_score
- grading_summary
- ai_confidence
- teacher_confirmed
- confirmed_by
- confirmed_at
```

说明：

- `suggested_score` 是 AI 建议分。
- `final_score` 是教师确认后的最终分。
- 如果教师未确认，不应作为正式成绩输出。

### 10.3 ErrorFinding

错误发现。

```text
ErrorFinding
- id
- grading_result_id
- error_type_id
- description
- related_text_span
- severity: low / medium / high
- confidence
- evidence_id
```

### 10.4 Evidence

证据对象。

```text
Evidence
- id
- evidence_type: image_region / recognized_text / rubric_point / teacher_comment / ai_reasoning
- source_object_type
- source_object_id
- content
- confidence
- created_at
```

### 10.5 AbilityDiagnosis

能力诊断。

```text
AbilityDiagnosis
- id
- student_id
- ability_point_id
- assignment_id
- question_id
- diagnosis_level: strong / normal / weak / unknown
- score
- confidence
- evidence_ids
- generated_by: system / ai / teacher
- teacher_confirmed
```

### 10.6 KnowledgeDiagnosis

知识点诊断。

```text
KnowledgeDiagnosis
- id
- student_id
- knowledge_point_id
- assignment_id
- question_id
- mastery_level: mastered / unstable / weak / unknown
- score
- confidence
- evidence_ids
```

### 10.7 ClassDiagnosis

班级诊断。

```text
ClassDiagnosis
- id
- class_id
- assignment_id
- diagnosis_type: question / ability / knowledge / error_type / student_group
- summary
- key_findings
- evidence_ids
- generated_at
```

### 10.8 StudentDiagnosis

学生个体诊断。

```text
StudentDiagnosis
- id
- student_id
- time_range
- subject_id
- summary
- strengths
- weaknesses
- evidence_ids
- recommended_actions
```

### 10.9 StudentLongitudinalProfile

学生长期画像。

```text
StudentLongitudinalProfile
- id
- student_id
- subject_id
- time_span_start
- time_span_end
- ability_trends
- knowledge_trends
- stable_strengths
- persistent_weaknesses
- recent_changes
- evidence_ids
- confidence
- last_updated_at
```

说明：

- 学生画像不应只停留在一次作业。
- 系统需要支持至少跨学期、跨学年的连续追踪。
- 对初中场景，应预留三年连续教学数据的建模能力。

### 10.10 ReviewPolicy

复核策略。

```text
ReviewPolicy
- id
- name
- target_type: grading_result / recognized_answer / diagnosis / feedback
- low_confidence_threshold
- auto_hide_threshold
- conflict_threshold
- sampling_rate
- require_chief_reviewer
- created_by
```

说明：

- 低于阈值的结果优先展示给教师复核。
- 高于阈值的结果可以默认折叠，但不能删除证据。
- 可支持抽样复核，避免教师被大量高置信度结果淹没。

### 10.11 ReviewQueue

复核队列。

```text
ReviewQueue
- id
- review_policy_id
- target_object_type
- target_object_id
- priority
- reason: low_confidence / model_conflict / teacher_conflict / random_sample / sensitive_output
- assigned_reviewer_id
- status: pending / reviewed / skipped / escalated
- created_at
- reviewed_at
```

### 10.12 ReviewConflict

复核冲突。

```text
ReviewConflict
- id
- target_object_type
- target_object_id
- reviewer_result_ids
- conflict_type: score_gap / rubric_disagreement / text_recognition_disagreement / diagnosis_disagreement
- conflict_summary
- chief_reviewer_id
- final_decision
- resolved_at
```

说明：

- 支持类似“多位教师批改结果不一致，需要主批确认”的机制。
- 该机制也可用于多个 AI 模型输出不一致时的主审流程。

## 11. 教学行动对象

### 11.1 Intervention

教学干预建议。

```text
Intervention
- id
- target_type: class / group / student
- target_id
- source_diagnosis_id
- intervention_type: review_lesson / remedial_exercise / individual_tutoring / parent_communication / next_lesson_adjustment
- title
- content
- priority: low / medium / high
- status: suggested / accepted / rejected / completed
```

### 11.2 ReviewLessonPlan

讲评课方案。

```text
ReviewLessonPlan
- id
- assignment_id
- class_diagnosis_id
- title
- objectives
- key_errors
- teaching_flow
- examples
- practice_items
```

### 11.3 RemedialExercise

补救练习。

```text
RemedialExercise
- id
- source_diagnosis_id
- target_ability_point_ids
- target_knowledge_point_ids
- exercise_content
- difficulty_level
- target_group
```

### 11.4 ParentCommunicationDraft

家校沟通草稿。

```text
ParentCommunicationDraft
- id
- student_id
- source_diagnosis_id
- tone: neutral / encouraging / serious
- content
- teacher_confirmed
```

原则：

家校沟通内容必须教师确认后才能使用。

## 12. AI 运行记录对象

### 12.1 AIJob

AI 任务。

```text
AIJob
- id
- job_type: ocr / answer_recognition / grading / diagnosis / lesson_generation / intervention_generation
- input_object_type
- input_object_id
- status: pending / running / success / failed / reviewed
- model_id
- prompt_template_id
- created_at
- finished_at
```

### 12.2 AIModel

AI 模型记录。

```text
AIModel
- id
- provider
- model_name
- model_version
- capability: text / vision / embedding / rerank
- usage_policy
```

### 12.3 PromptTemplate

提示词模板。

```text
PromptTemplate
- id
- name
- version
- task_type
- content
- input_schema
- output_schema
```

### 12.4 AIOutput

AI 输出。

```text
AIOutput
- id
- ai_job_id
- raw_output
- parsed_output
- confidence
- error_message
- token_usage
- cost_estimate
```

### 12.5 HumanReview

人工审核。

```text
HumanReview
- id
- target_object_type
- target_object_id
- reviewer_id
- review_action: confirm / correct / reject / comment
- before_value
- after_value
- comment
- reviewed_at
```

## 13. 关系对象模型

### 13.1 GraphRelation

统一关系对象。

```text
GraphRelation
- id
- from_type
- from_id
- to_type
- to_id
- relation_type
- weight
- confidence
- evidence_ids
- source: manual / imported / ai_inferred / system
- valid_from
- valid_to
- created_at
- updated_at
```

### 13.2 关系类型

人员关系：

```text
Teacher teaches Class
Teacher has_role TeacherRole
Class contains Student
Student belongs_to StudentGroup
```

教材关系：

```text
Textbook contains BookVolume
BookVolume contains Unit
Unit contains Lesson
Lesson contains LessonSegment
Lesson aligns_to CurriculumStandard
```

知识图谱关系：

```text
KnowledgePoint prerequisite_of KnowledgePoint
KnowledgePoint related_to KnowledgePoint
KnowledgePoint part_of KnowledgePoint
AbilityPoint prerequisite_of AbilityPoint
AbilityPoint related_to KnowledgePoint
QuestionType requires AbilityPoint
TextGenre uses QuestionType
```

题目关系：

```text
Question based_on QuestionMaterial
Question assesses KnowledgePoint
Question assesses AbilityPoint
Question uses QuestionType
Question has_rubric Rubric
Rubric contains RubricCriterion
```

作答关系：

```text
Student submits Submission
Submission contains AnswerImage
AnswerImage produces RecognizedAnswer
RecognizedAnswer answers Question
GradingResult based_on RecognizedAnswer
GradingResult based_on RubricCriterion
ErrorFinding points_to ErrorType
ErrorFinding evidenced_by Evidence
```

诊断关系：

```text
AbilityDiagnosis derived_from GradingResult
KnowledgeDiagnosis derived_from GradingResult
ClassDiagnosis aggregates GradingResult
StudentDiagnosis aggregates AbilityDiagnosis
Intervention responds_to Diagnosis
```

## 14. 初中语文专项模型

### 14.1 语文作业类型

```text
reading_comprehension      现代文阅读
classical_chinese          文言文阅读
poetry_appreciation        古诗词鉴赏
language_application       语言运用
writing                    作文
dictation                  默写
unit_test                  单元测验
```

### 14.2 语文能力维度

```text
信息提取
内容概括
词句理解
语言赏析
结构分析
人物形象分析
主旨理解
文本探究
文言翻译
诗歌意象理解
表达组织
审题能力
```

### 14.3 语文题目到能力的映射

示例：

```text
概括题 -> 信息提取 + 内容概括 + 表达组织
赏析题 -> 语言赏析 + 文本依据 + 表达组织
作用题 -> 结构分析 + 内容理解 + 主旨理解
人物形象题 -> 信息提取 + 人物分析 + 文本依据
```

### 14.4 语文错误类型

```text
漏答要点
答非所问
只摘抄原文
没有结合文本
概括不准确
表述空泛
术语误用
逻辑混乱
层次不清
情感理解偏差
主旨理解偏差
文言关键词误解
```

### 14.5 语文评分特点

语文评分应支持：

```text
关键词命中
语义等价
答题角度完整性
文本依据充分性
表达清晰度
教师人工裁量
```

因此不能用纯对错模型，应采用：

```text
评分点匹配 + 能力诊断 + 错误类型识别 + 教师确认
```

## 15. 核心数据流

### 15.1 教师配置流

```text
教师创建 TeachingScope
-> 选择学期、年级、学科、教材、班级、是否班主任
-> 系统确定后续作业和教材范围
```

### 15.2 作业创建流

```text
教师创建 Assignment
-> 添加 Question
-> 添加 QuestionMaterial
-> 添加 Rubric / RubricCriterion
-> 绑定 KnowledgePoint / AbilityPoint / QuestionType
```

### 15.2.1 题目电子化流

```text
题目图片 / 电子文档
-> 识别题干、材料、参考答案、评分标准
-> 生成 QuestionBankItem
-> 教师编辑确认
-> 发布为可复用电子题目
-> 被 Assignment 引用
```

### 15.3 作业采集流

```text
学生整页作业图片 AnswerImage
-> 页面边缘识别与质量检查
-> 版面结构识别
-> AnswerRegion 裁剪
-> 同一学生多页/多区域合并
-> 题目与作答区域匹配
-> OCR/多模态识别 RecognizedAnswer
-> 教师校对 TeacherCorrection
-> 形成可信作答文本
```

### 15.4 批改诊断流

```text
RecognizedAnswer + Question + Rubric
-> AI 初评 GradingResult
-> ErrorFinding
-> TeacherReview
-> AbilityDiagnosis / KnowledgeDiagnosis
```

### 15.5 教学行动流

```text
ClassDiagnosis / StudentDiagnosis
-> Intervention
-> ReviewLessonPlan / RemedialExercise / ParentCommunicationDraft
```

## 16. 权限与隐私初稿

### 16.1 隐私等级

```text
public             公开信息
internal           系统内部信息
sensitive          学生姓名、作业、成绩
highly_sensitive   家庭信息、心理状态、特殊标记
```

### 16.2 教师权限

```text
任课教师：查看自己任教班级和作业
班主任：查看班级综合情况
管理员：管理基础数据
```

阶段策略：

```text
第一阶段优先实现任课教师工作流。
班主任权限保留模型扩展能力，主要用于后续家校沟通、班级综合画像和德育事务。
```

### 16.3 原则

```text
学生画像不得无证据生成
高风险判断不得自动通知家长
AI 生成的正式反馈必须教师确认
```

## 17. MVP 切片建议

本文档不直接限定 MVP，但建议后续 MVP 从完整模型中切一条闭环：

```text
TeachingScope
Class / Student
Textbook / Unit / Lesson
QuestionBankItem
Question / Rubric
AnswerImage / AnswerRegion / RecognizedAnswer
GradingResult
ClassDiagnosis
StudentLongitudinalProfile
ReviewLessonPlan
```

即：

```text
一个教师
一个学期范围
一个班级
一份初中语文作业
若干学生作业图片
整页上传和裁剪识别
OCR/多模态识别
AI 初评
教师确认
班级诊断
学生长期画像更新
讲评建议
```

说明：

- 第一阶段可以只展示一次作业分析结果，但数据结构必须写入长期画像。
- 拍照入口应优先考虑移动端，同时网页端也要支持上传。
- 家校沟通先不进入第一阶段功能闭环，但保留权限和对象扩展。

## 18. 本轮评审结论

以下结论来自 2026-07-03 产品评审，已纳入当前模型。

### 18.1 学科范围

第一阶段明确以初中语文为主。

其他学科不进入第一阶段实现，但领域模型需要保留扩展能力。

### 18.2 作业图片采集

第一阶段优先支持整页上传。

处理顺序：

```text
整页图片
-> 页面边缘识别
-> 裁剪和质量校正
-> 同一学生多页合并
-> 题目和作答区域识别
-> OCR/多模态识别
```

不要求教师一开始按题裁剪上传。

### 18.3 学生名单

第一阶段同时支持：

```text
手动录入
Excel 导入
学生信息编辑
学生删除/归档
字段扩展
```

### 18.4 题目电子化

第一阶段支持：

```text
题目图片识别
电子版本导入
参考答案识别
评分标准识别
```

识别后的题目应作为 `QuestionBankItem` 存在，后续可复用、编辑和关联作业。

### 18.5 教材和课文内容

允许存储教材和课文内容。

检索方式需要支持：

```text
关键词触发
全文检索
语义检索
混合检索
```

可参考 skill 的触发方式：由少量关键词或意图启动对应资料检索。

### 18.6 教师复核机制

教师确认不应只做逐题确认。

应支持：

```text
按置信度从低到高排序
高置信度结果默认折叠或隐藏
低置信度结果优先进入 ReviewQueue
多人或多模型结果不一致时进入 ReviewConflict
必要时由主批教师确认最终结果
```

### 18.7 学生画像

学生画像从第一阶段开始就应写入长期模型。

第一阶段可以只展示有限画像，但底层结构必须支持长期追踪，目标是至少覆盖初中三年连续学习过程。

### 18.8 家校沟通

家校沟通不进入第一阶段核心功能。

但需要保留：

```text
班主任角色
家校沟通草稿对象
权限扩展
学生长期画像到沟通内容的引用能力
```

### 18.9 教师角色与权限

第一阶段先做任课教师工作流。

班主任权限保留扩展能力，主要用于后续家校沟通、班级综合画像和德育事务。

### 18.10 终端形态

第一阶段需要同时考虑：

```text
移动端拍照流程
网页端上传流程
```

移动端拍照是作业采集的关键入口，网页端适合管理、校对、分析和讲评内容编辑。

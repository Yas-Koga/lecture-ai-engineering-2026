# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：素のLLMでは、一般的な回答を行っているが、エージェントの回答では、AIE基礎講座の内容に沿った回答となっている。例えば、用語集にない用語の質問（LecNNタグとは何か）をした場合、素のLLMでは「AIE2026の講座で使用される**「lecNN タグ」**（NNには数字が入ります。例：`lec01`, `lec02`, `lec15` など）は、**各回の講義（Lecture）を識別するための管理用タグ（キーワード）**です。」といった回答となったが、エージェントの回答では、「用語集にはない言葉です。講座の資料や公式な用語として定義されていない可能性があります。どのような文脈で出てきた言葉か教えていただければ、推測できるかもしれません。」といったより事実に沿った回答が得られた。
- `--trace` で見えたツール呼び出し（ツール名と引数）：
"AIE2026 の講座の宿題にある「3レベル」とは何ですか? "（質問１）の質問をした際のツールの呼び出しは、search_glossaryが１回呼び出されており、引数は'3レベル’であった。また、"LoRA とQLoRAの違いは?"（質問２）の質問をしたときは、serach_glossaryが2回予備さだれており、引数は、それぞれ’LoRA’と’QLoRA’であった。
- 1つの質問での LLM 呼び出し回数（`requests`）：質問１、質問２のいずれもLLMは2回呼び出されている。
- 追加したツールと、docstring に書いた説明：tools.pyにmake_quiz(lecture: int) -> str:を追加し、docstringには、 「指定した回の用語から 1つを選び、確認問題と答えを返す。「第2回の確認問題を出して」のように、復習用の問題を求められたときに使う。」のように追加した。agent.pyにmake_quizツールを登録した。

## 宿題

- 取り組んだレベル：基礎
- やったこと：
app/agent.pyのINSTRUCTIONやapp\toolsのdocstringを(1)～(3)のように変更して、"LoRA とQLoRAの違いは?"の質問を行った。
(1)app/agent.pyのINSTRUCTIONから「必ずsearch_glossaryで用語集を確認する」の行を削除した。
(2)app/tools.pyのdocstringの「用語を一つだけ入れる」から「まとめて入れる」に変更した。
(3)app/agent.pyのINSTRUCTIONをINSTRUCTIONの行を「あなたはAIエンジニアリング講座（AIE2026）の学習支援エージェントです。」のみにした。
- 結果：
(1)行の削除の前後で回答に変化なし。
(2)内容変更の前後で回答には大きな変化はないが、ツールコールが2回から1回に減少している。
(3)回答の形式には大きな変化がみられるが、内容的には大きな変化は見られず、ツールコールの回数も2回と変化なし。
- 気づき・疑問：INSTRUCTIONを多少変更しても、search_glossaryが呼び出されないということはなく、用語集は呼び出されていた。tools.pyのdocstringを変更した際には、実行をトレースした結果が変わっているが、本来、docstringにはコードの実行とはかかわりがないはずであるのに、なぜ、トレース結果が変わっているのか疑問に思った。


## 宿題メモ

## INSTRUCTIONの行を「あなたはAIエンジニアリング講座（AIE2026）の学習支援エージェントです。」のみにした
## uv run python -m app.agent --trace "LoRA とQLoRAの違いは?"
[model] gemini-3.5-flash-lite
本講座（AIE2026）の用語集を基に、**LoRA** と **QLoRA** の違いを解説します。

---

### **LoRA（Low-Rank Adaptation）**
* **仕組み**: 事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、**その小さな行列だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法**です。
* **特徴**: 
  * 学習するパラメータ数が大幅に少なくなる。
  * 学習結果（アダプタ）を小さなファイルサイズで保存できる。
  * 第2回の演習では、モデルがGPUに載るためこちらをメインに扱います。

### **QLoRA（Quantized LoRA）**
* **仕組み**: ベースモデルの重みを **4ビットなどに量子化してメモリを大幅に節約** したうえで、**LoRAの仕組みを使って学習する手法**です。
* **特徴**:
  * メモリ消費量を劇的に減らせるため、**より大きなモデルを小さなGPU（VRAMが少ない環境）でも扱える**ようになります。
  * ※本講座の第2回では、モデルを4ビットで読み込んだ際のメモリ量や出力の違いを発展内容として比較します。

---

### **まとめ**
* **LoRA**: 基本の「低ランク行列による効率的な微調整」手法。
* **QLoRA**: LoRAに**「モデルの量子化（4ビット化）」**を組み合わせることで、さらに省メモリで巨大モデルの学習を可能にした発展手法。

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0013363'), details={'text_prompt_tokens': 1146}, input_text_tokens=1146, input_tokens=1146, output_tokens=397, requests=2, tool_calls=2)


## docstringの用語を一つだけ入れるからまとめて入れるに変更した。
## uv run python -m app.agent --trace "LoRA とQLoRAの違いは?"の実行結果
[model] gemini-3.5-flash-lite
LoRA（Low-Rank Adaptation）とQLoRA（Quantized LoRA）の主な違いは、**ベースモデルの重みを量子化しているかどうか**、およびそれによる**メモリ効率**です。

* **LoRA**: 
  事前学習済みのモデルの重み（元の精度、例えば16ビットなど）をそのまま固定し、追加の小さな低ランク行列（アダプタ）だけを学習することで、効率的にファインチューニングを行う手法です。

* **QLoRA**: 
  LoRAの仕組みをベースにしつつ、ベースモデルの重みを**4ビットなど極限まで量子化（軽量化）**してメモリ消費を大幅に抑えたうえで、LoRAによる学習を行う手法です。これにより、より大きなモデルをより少ないGPUメモリで動かして学習できるようになります。

第2回（第2回演習など）
（参考用語：LoRA、QLoRA、PEFT、量子化）

--- trace ---
[tool call]   search_glossary({'query': 'LoRA QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[usage] RunUsage(cost=Decimal('0.0008769'), details={'text_prompt_tokens': 1023}, input_text_tokens=1023, input_tokens=1023, output_tokens=228, requests=2, tool_calls=1)

## INSTRUTIONの必ず検索するの行を削除した
## uv run python -m app.agent --trace "LoRA とQLoRAの違いは?"の実行結果
[model] gemini-3.5-flash-lite
**LoRA**と**QLoRA**の主な違いは、ベースモデルの重みをそのまま使うか、量子化してメモリを大幅に削減するかにあります。

* **LoRA（Low-Rank Adaptation）**
  事前学習済みの重みは固定したまま、重みの更新分を小さな低ランク行列（2つの小さな行列の積）で表し、その部分だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習パラメータを少なく抑えられます。

* **QLoRA（Quantized LoRA）**
  ベースモデルの重みを4ビットなどに量子化（Quantization）してメモリをさらに節約したうえで、LoRAによるファインチューニングを行う手法です。これにより、より大きなモデルを小さなGPUでも効率よく扱えるようになります。

（第2回）

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0009375'), details={'text_prompt_tokens': 1300}, input_text_tokens=1300, input_tokens=1300, output_tokens=219, requests=2, tool_calls=2)

## 最初の実行結果
## uv run python -m app.agent --trace "LoRA とQLoRAの違いは?"
[model] gemini-3.5-flash-lite
**LoRA**と**QLoRA**の主な違いは、ベースモデルの重みをそのまま使うか、量子化してメモリを大幅に削減するかにあります。

* **LoRA（Low-Rank Adaptation）**
  事前学習済みの重みは固定したまま、重みの更新分を小さな低ランク行列（2つの小さな行列の積）で表し、その部分だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習パラメータを少なく抑えられます。

* **QLoRA（Quantized LoRA）**
  ベースモデルの重みを4ビットなどに量子化（Quantization）してメモリをさらに節約したうえで、LoRAによるファインチューニングを行う手法です。これにより、より大きなモデルを小さなGPUでも効率よく扱えるようになります。

（第2回）

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0009375'), details={'text_prompt_tokens': 1300}, input_text_tokens=1300, input_tokens=1300, output_tokens=219, requests=2, tool_calls=2)


## 演習メモ
## uv run python -m app.ask "AIE2026 の講座の宿題にある「3レベル」とは何ですか? "の実行結果
[model] gemini-3.5-flash-lite
[質問] AIE2026 の講座の宿題にある「3レベル」とは何ですか? 

[tokens] 入力: 23, 出力: 757
AIE2026（おそらくAI関連の講座やプログラム）の宿題における「3レベル」という言葉ですね。

コンテキスト（具体的な講座名や直前の授業内容）が分からないため確実ではありませんが、AIやプログラミング、ビジネスの講座で「3レベルの宿題・課題」が出た場合、一般的に以下のいずれかの分類を指している可能性が高いです。

ご自身の受講している内容に一番近いものを確認してみてください。

---

### 1. 難易度による分類（よくあるパターン）
多くの実践的なAI講座では、受講生のスキルに合わせて3段階の難易度が用意されています。

*   **レベル1（初級・基礎編）：**
    *   **内容:** チュートリアルの通りに手を動かす、指定されたプロンプトをそのまま入力する、穴埋め式のコードを書くなど。
    *   **目的:** ツールや概念の基本操作を覚える。
*   **レベル2（中級・応用編）：**
    *   **内容:** 少し条件を変えてみる、自分の業務や興味に合わせてプロンプトをカスタマイズする、エラーを自分で調べて解決するなど。
    *   **目的:** 知識を実用的な形に応用する。
*   **レベル3（上級・発展編）：**
    *   **内容:** 完全なゼロから自分で作成する、複数のAIツールを組み合わせる、複雑な課題に対して自力で仮説検証を行うなど。
    *   **目的:** 応用力をつけ、自走できるようになる。

*(※講座によっては、全員が「レベル1」から順にすべてやる必要がある場合と、「自分のレベルに合わせてどれかを選ぶ（または選択式）」という場合があります)*

---

### 2. 生成AI活用の習熟度による分類
AIスクールなどで、AIを使いこなすステップとして以下のような3レベルが定義されている場合もあります。

*   **レベル1：指示通りに使う（作業の効率化）**
    *   文章の要約、壁打ち、簡単なメール作成など。
*   **レベル2：プロセスを自動化・高度化する（ワークフローの構築）**
    *   複数のプロンプトを連係させる、カスタムGPTを作るなど。
*   **レベル3：システムや事業に組み込む（イノベーション）**
    *   APIを使った連携、プロダクト開発、業務プロセスの根本的な変革など。

---

### 3. 提出要件やスコープ（範囲）のレベル
*   **必須（マスト）：** 全員が必ずクリアしなければならない基本課題。
*   **推奨（努力義務）：** 時間がある人、より深く学びたい人が取り組む応用課題。
*   **挑戦（チャレンジ）：** さらに難易度の高いボーナス課題。

---

### 💡 確実な確認方法
もし上記に当てはまらない場合や不安な場合は、以下の方法で確認することをおすすめします。

1.  **宿題の指示書（Notion、スプレッドシート、LMSなど）をもう一度見直す:**
    *   「レベル1」「レベル2」「レベル3」の具体的な要件が書いていないか探す。
2.  **同期の受講生やコミュニティ（DiscordやSlackなど）で聞いてみる:**
    *   「今回の宿題の『3レベル』って、どういう意味ですか？」と他の人に聞くと、すぐに教えてもらえることが多いです。


## uv run python -m app.agent --trace "AIE2026 の講座の宿題にある「3レベル」とは何ですか? "の実行結果
[model] gemini-3.5-flash-lite
「3レベル」とは、AIE2026の宿題の構成を表す仕組みのことです。

この講座の宿題は、以下の3つの段階（レベル）に分かれています。

1. **基礎**：講義内容の確認（知識の定着を図るレベル）
2. **実装**：自分でコードを書く（手を動かして理解するレベル）
3. **発展**：調査や試行（より深く応用的なことに挑むレベル）

受講にあたっては、この3つのうち**どれか1つ**に取り組めば、その回の宿題は完了となります。原則として採点は行われませんので、ご自身の理解度や目的に合わせて選んでみてください。

関連する回：第1回

--- trace ---
[tool call]   search_glossary({'query': '3レベル'})
[tool return] search_glossary -> [{"term": "宿題の3レベル", "lecture": 1, "definition": "AIE2026の宿題の構成。「基礎（講義内容の確認）」「実装（自分でコードを書く）」「発展（調査や試行）」の3段階があり、どれか1つに取り組…
[usage] RunUsage(cost=Decimal('0.0006911'), details={'text_prompt_tokens': 787}, input_text_tokens=787, input_tokens=787, output_tokens=182, requests=2, tool_calls=1)


## uv run python -m app.agent --trace "LoRA とQLoRAの違いは?"の実行結果
[model] gemini-3.5-flash-lite
LoRAとQLoRAの違いは、ベースモデルの重みをそのまま扱うか、量子化してメモリを節約するかにあります。

* **LoRA**: 事前学習済みの重みを固定したまま、重みの更新分を2つの小さな低ランク行列の積として表現し、その部分だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習パラメータやファイルサイズを大幅に削減できます。
* **QLoRA**: LoRAの仕組みをベースにしつつ、さらにベースモデルの重みを4ビットなどに**量子化**してメモリ消費量を極限まで抑えたうえで学習を行う手法です。これにより、より大きなモデルを小さなGPUメモリでも効率よく扱えるようになります。

関係する回：第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0008613'), details={'text_prompt_tokens': 1196}, input_text_tokens=1196, input_tokens=1196, output_tokens=201, requests=2, tool_calls=2)



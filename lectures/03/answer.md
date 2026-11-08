# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
第1回の場合は質問に対して、単にLLMが応答していたが、第3回のエージェントの場合、LLMへの指示が、INSTRUCTIONS やツールに記載された内容に従って動作していた。

【出力結果：素のLLMの場合】
un python -m app.agent "AIE2026の講座の宿題にある「3レベル」とは何ですか？"
[model] gemini-3.5-flash-lite
「3レベル」とは、AIE2026の宿題の構成を表す仕組みです。

宿題には以下の3段階のレベルが用意されており、受講生は**ご自身の学習目的に合わせてどれか1つを選択して取り組む**ことで、その回の宿題が完了となります（原則として採点は行いません）。

1. **基礎**：講義内容の確認を目的とした問題
2. **実装**：自分で実際にコードを書く問題
3. **発展**：より高度な調査や試行を行う問題

自分の理解度や使える時間に合わせて選んで進めてみてくださいね。

関係する回：第1回

【出力結果：エージェントの場合】
(aie2026-exercises) @Teraoka-Hiroki ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
[model] gemini-3.5-flash-lite
講座の用語集によると、LoRAとQLoRAの違いは以下のとおりです。

* **LoRA**
  事前学習済みの重みを固定したまま、重みの更新分を2つの小さな低ランク行列の積で表し、その小さな行列だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習するパラメータが少なく、学習結果（アダプタ）も小さなファイルで保存できます。

* **QLoRA**
  ベースモデルの重みを4ビットなどに量子化（Quantization）してメモリを大きく節約したうえで、LoRAによる学習を行う手法です。これにより、より大きなモデルを比較的小さなGPUで扱えるようになります。

関係する回：第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0008513'), details={'text_prompt_tokens': 1196}, input_text



- `--trace` で見えたツール呼び出し（ツール名と引数）：
(aie2026-exercises) @Teraoka-Hiroki ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --list-tools
モデルに見えているツール: 3 個

[search_glossary]
  講座の用語集から用語を検索し、定義と、どの回で扱うかを返す。
  
  AIエンジニアリングの用語や、この講座（AIE2026）に固有の言葉について
  答える前に使うこと。query には調べたい用語を1つだけ入れる（例: "LoRA"）。
  引数: {"query": {"type": "string"}}

[get_lecture_summary]
  指定した回（1始まりの整数）の講義の要点を返す。
  
  「第3回では何を学ぶ？」のように、回の内容や関係を聞かれたときに使う。
  引数: {"lecture": {"type": "integer"}}

[make_quiz]
  指定した回の用語から1つ選び、意味を問う確認問題と答えを返す。
  
  「第2回の確認問題を出して」のように、特定の回の用語で復習したいと
  頼まれたときに使う。
  引数: {"lecture": {"type": "integer"}}


- 1つの質問での LLM 呼び出し回数（`requests`）：
複数回の呼び出しがあった。
--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…

- 追加したツールと、docstring に書いた説明：
クイズの作成ツールを追加した。
def make_quiz(lecture: int) -> str:
    """指定した回の用語から1つ選び、意味を問う確認問題と答えを返す。

    「第2回の確認問題を出して」のように、特定の回の用語で復習したいと
    頼まれたときに使う。
    """
    terms = [term for term in load_glossary() if term["lecture"] == lecture]
    if not terms:
        return f"第{lecture}回の用語が用語集にありません。"

    term = random.choice(terms)
    return f"問題: 「{term['term']}」の意味は？\n答え: {term['definition']}"

docstring に書いた説明：
「- 用語集に見つからないことは、推測で断定せず「用語集にはない」と伝える。」に続けて、
以下の指示を加えた。
「その場合は、「用語集にはない」と答えたのち、あなたの知識での説明を提供する。」


## 宿題

- 取り組んだレベル：基礎

- やったこと：
①docstring に書いた説明：
「- 用語集に見つからないことは、推測で断定せず「用語集にはない」と伝える。」に続けて、
以下の指示を加えた。
「その場合は、「用語集にはない」と答えたのち、あなたの知識での説明を提供する。」

➁docstring の「用語を1つだけ入れる」を「まとめて入れる」に変える

- 結果：
①(aie2026-exercises) @Teraoka-Hiroki ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "バイブコーディングとは何ですか？"
[model] gemini-3.5-flash-lite
用語集にはない言葉ですが、一般的なAIエンジニアリングの文脈において「バイブコーディング（Vibe Coding）」とは、人間が手動で1行ずつコードを書くのではなく、AI（LLMやコード生成エージェント）に大まかな意図や「ノリ・雰囲気（Vibe）」を伝え、AI側にコードの大部分を記述・修正してもらいながら開発を進める新しいスタイルを指します。

人間はプログラマーというよりは「ディレクター」や「レビュアー」のような役割を担い、対話を通じてプロダクトを作り上げていくのが特徴です。

関係する回：第1回、第12回

--- trace ---
[tool call]   search_glossary({'query': 'バイブコーディング'})
[tool return] search_glossary -> 用語集に「バイブコーディング」は見つかりませんでした。…
[usage] RunUsage(cost=Decimal('0.0006751'), details={'text_prompt_tokens': 892}, input_text_tokens=892, input_tokens=892, output_tokens=163, requests=2, tool_calls=1)


➁(aie2026-exercises) @Teraoka-Hiroki ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
[model] gemini-3.5-flash-lite
LoRAとQLoRAの違いは、ベースモデルの重みをそのまま使うか、量子化してメモリを大幅に節約するかという点にあります。

* **LoRA（Low-Rank Adaptation）**
  事前学習済みの重みを固定したまま、更新分を表す小さな行列（低ランク行列）だけを追加して学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習パラメータが非常に少なく、軽量にファインチューニングを行えます。

* **QLoRA（Quantized LoRA）**
  LoRAの仕組みをベースにしつつ、ベースモデルの重みを4ビットなどの低ビットに量子化して読み込む手法です。これにより必要なVRAM（GPUメモリ）を大幅に削減し、より巨大なモデルを限られたハードウェア環境で効率よく学習できるようになります。

第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[usage] RunUsage(cost=Decimal('0.0008304'), details={'text_prompt_tokens': 1093}, input_text_tokens=1093, input_tokens=1093, output_tokens=201, requests=2, tool_calls=1)


- 気づき・疑問：エージェントは指示に正確に答えることが分かった。それゆえ、人間の指示が正確で的確であることが重要だと分かった。

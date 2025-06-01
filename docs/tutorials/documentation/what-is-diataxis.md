---
tags:
  - Diataxis
  - Documentation
  - Tutorials
---

# What is Diátaxis

[Diátaxis](https://diataxis.fr/) (ダイアタキシス) とは、文書化に対する考え方やアプローチを体系化したもの。
Ubuntu を開発・提供している Canonical 社も、[ドキュメント管理に Diátaxis を採用している](https://ubuntu.com/blog/moving-toward-diataxis)とのこと。

## Diátaxis の構成

公式ドキュメントの以下の図によると、横軸に **習得 (Acquisition)** と **応用 (Application)**、
縦軸に **認知 (Cognition)** と **実践 (Action)** をとった四象限に分類されるとのこと。

![The Diátaxis map](https://diataxis.fr/_images/diataxis.png)

[The Diátaxis compass](https://diataxis.fr/start-here/#the-diataxis-compass) にあるとおり、

*   (上部の) **チュートリアル (Tutorials)** と **ハウツーガイド (How-to Guides)** は、
    読み手が「どのようにするか (実践、Action)」 に焦点を当てている

*   (下部の) **説明 (Explanation)** と **参照 (Reference)** は、
    読み手が「それはどのようなものか (認知、Cognition)」 に焦点を当てている

その一方で、

*   (左側の) **チュートリアル (Tutorials)** と **説明 (Explanatin)** は、
    「スキルの習得 (Acquisitin、読み手の学習)」 に役立つ

*   (右側の) **ハウツーガイド (How-to Guides)** と **リファレンス (Reference)** は、
    「スキルの応用 (Application、読み手の作業)」 に役立つ

構成となっている。

整理すると、以下の表のとおり。

| ドキュメントが             | 読み手にとっての               | それは以下を読むと良いだろう   |
|----------------------------|--------------------------------|--------------------------------|
| 行動 (Action) を学ぶもので | スキルの習得を目指しているなら | チュートリアル (Tutorial)      |
| 行動 (Action) を学ぶもので | スキルの応用を目指しているなら | ハウツーガイド (How-to Guides) |
| 情報を認知 (Cognision) し  | 知識の習得を目指しているなら   | 説明 (Explanation)             |
| 情報を認知 (Cognision) し  | 知識の応用を目指しているなら   | 参照 (Reference)               |


## Diátaxis の適用

[Expectations and guidance](https://diataxis.fr/map/#expectations-and-guidance) によれば、
それぞれの象限の文書は以下のような内容を含むとのこと。

|  | Tutorials | How-to guides | Reference | Explanation |
|---|---|---|---|---|
| **what they do**<br />(文書が何をするか) | introduce, educate, lead<br />(紹介する、教育する、指導する) | guide<br />(ガイド) | state, describe, inform<br />(述べる、説明する、知らせる) | explain, clarify, discuss<br />(説明する、明確にする、議論する) |
| **answers the question**<br />(文書が答える質問は) | “Can you teach me to…?”<br />(教えてくれますか？) | “How do I…?”<br />(どうやってすれば良いですか？) | “What is…?”<br />(それは何ですか？) | “Why…?”<br />(何故ですか？) |
| **oriented to**<br />(文書が重視するものは) | learning<br />(学び) | goals<br />(達成) | information<br />(情報) | understanding<br />(理解) |
| **purpose**<br />(文書の目的は) | to provide a learning experience<br />(学習経験を提供する) | to help achieve a particular goal<br />(特定の目標を達成する) | to describe the machinery<br />(機械的な説明) | to illuminate a topic<br />(主題を明らかにする) |
| **form**<br />(文書の形式は) | a lesson<br />(レッスン) | a series of steps<br />(一連の手順) | dry description<br />(簡潔で事実の説明) | discursive explanation<br />(広範囲にわたるとりとめのない説明) |
| **analogy**<br />(文書の例) | teaching a child how to cook<br />(子供に料理の仕方を教える) | a recipe in a cookery book<br />(料理本のレシピ) | information on the back of a food packet<br />(食品パッケージの裏面情報) | an article on culinary social history<br />(料理社会史に関する記事) |

筆者なりの解釈としては、以下のような分類になるかと感じる。

**Tutorials**
:   「Spring boot とは」とか「GitHub Actions を使った CI pipeline の作り方」のような、
    主題を全く経験したことのない人に教えるような内容。

**How-to Guides**
:   「まっさらな AWS 環境に EKS クラスタを構築するセットアップ手順書」のように、
    (筆者としては、) Tutorials よりもっと端的な、余計な情報を含んでいない最短ステップで達成可能な手順書など。

**Reference**
:   アーキテクチャ設計書、API 仕様書、CLI manpage、など、仕様書に分類されるもの。

**Explanation**
:   「オブジェクト指向設計の原則の重要性について」や「マイクロサービスアーキテクチャが有用な構成」など、
    「**なぜ** そうするのか、 **なぜ** そうあるべきか」を応えるような記事や記述。
    もしくは、 ADR (Architecture Decision Records) のようなもの。

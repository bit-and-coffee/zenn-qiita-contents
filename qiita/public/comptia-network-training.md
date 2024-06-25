---
title: 【神速】chatGPTでWeb模擬試験アプリを作った話
tags:
  - Network
  - idea
  - OpenAI
  - ChatGPT
private: false
updated_at: '2024-06-26T07:57:44+09:00'
id: 535408cacc28046b7a9a
organization_url_name: null
---
## １.はじめに
前回CompTIA Network+の勉強法について記事を投稿しました。

https://zenn.dev/bit_and_coffee/articles/67b9bf52558904

記事でも書いた通り勉強リソースが少ないので、職場のメンバーが勉強できる環境を作れたらと言う気持ちから模擬試験アプリを作成しました。


## 2.模擬試験アプリの作り方
### 1. tldrawでHTMLを生成  
見た目については、[tldraw](https://tldraw.dev/)とういうwebアプリで作成しました。
書いた絵をHTMLに変換してくれます。調べると色々な説明がヒットするので調べてみてほしいです。

https://generativeinfo365.com/?p=2210#google_vignette

この機能を使うにはchatGPTのAPIキーが必要です。

https://ai-workstyle.com/gpt-apikey/

### 2. chatGPTにJavascript作成を依頼
チャット履歴はありませんが、以下の用に質問するとコードを生成してくれます。
```
模擬試験アプリ作成のため、以下の要件を満たすJavascriptのコードを作成してください。

１ 問題の表示（ページの初期化）
２ 次へ、戻るボタンで問題が前後する
３ 問題に紐づいた解答群を表示させる
４ 終了ボタンを押すと採点結果表示（表形式で）
５ CSV出力ボタンで採点結果をダウンロード
```
```javascript:Javascript
const questions = [
    {
        question: "問題1: 日本の首都はどこですか？",
        choices: ["東京", "大阪", "京都", "札幌"],
        correctAnswer: "東京"
    },
    {
        question: "問題2: 富士山の標高は？",
        choices: ["2776m", "3376m", "3776m", "4276m"],
        correctAnswer: "3776m"
    }
    // 他の問題を追加
];

let currentQuestionIndex = 0;
let answers = [];

document.addEventListener("DOMContentLoaded", () => {
    const questionElement = document.getElementById("question");
    const choicesElement = document.getElementById("choices");
    const prevBtn = document.getElementById("prev-btn");
    const nextBtn = document.getElementById("next-btn");
    const submitBtn = document.getElementById("submit-btn");
    const resultsContainer = document.getElementById("results-container");
    const downloadCsvBtn = document.getElementById("download-csv-btn");

    function showQuestion() {
        const currentQuestion = questions[currentQuestionIndex];
        questionElement.textContent = currentQuestion.question;
        choicesElement.innerHTML = '';
        currentQuestion.choices.forEach((choice, index) => {
            const choiceLabel = document.createElement("label");
            const choiceInput = document.createElement("input");
            choiceInput.type = "radio";
            choiceInput.name = "choice";
            choiceInput.value = choice;
            if (answers[currentQuestionIndex] === choice) {
                choiceInput.checked = true;
            }
            choiceLabel.appendChild(choiceInput);
            choiceLabel.appendChild(document.createTextNode(choice));
            choicesElement.appendChild(choiceLabel);
            choicesElement.appendChild(document.createElement("br"));
        });
    }

    function showResults() {
        resultsContainer.innerHTML = '<h2>採点結果</h2>';
        const table = document.createElement('table');
        const headerRow = table.insertRow();
        headerRow.insertCell().textContent = '問題';
        headerRow.insertCell().textContent = 'あなたの解答';
        headerRow.insertCell().textContent = '正解';
        questions.forEach((question, index) => {
            const row = table.insertRow();
            row.insertCell().textContent = question.question;
            row.insertCell().textContent = answers[index] || '未回答';
            row.insertCell().textContent = question.correctAnswer;
        });
        resultsContainer.appendChild(table);
        resultsContainer.style.display = 'block';
    }

    function downloadCSV() {
        let csvContent = "data:text/csv;charset=utf-8,";
        csvContent += "問題,あなたの解答,正解\n";
        questions.forEach((question, index) => {
            csvContent += `${question.question},${answers[index] || '未回答'},${question.correctAnswer}\n`;
        });
        const encodedUri = encodeURI(csvContent);
        const link = document.createElement("a");
        link.setAttribute("href", encodedUri);
        link.setAttribute("download", "results.csv");
        document.body.appendChild(link);
        link.click();
    }

    prevBtn.addEventListener("click", () => {
        currentQuestionIndex = Math.max(currentQuestionIndex - 1, 0);
        showQuestion();
    });

    nextBtn.addEventListener("click", () => {
        const selectedChoice = document.querySelector('input[name="choice"]:checked');
        if (selectedChoice) {
            answers[currentQuestionIndex] = selectedChoice.value;
        }
        currentQuestionIndex = Math.min(currentQuestionIndex + 1, questions.length - 1);
        showQuestion();
    });

    submitBtn.addEventListener("click", () => {
        const selectedChoice = document.querySelector('input[name="choice"]:checked');
        if (selectedChoice) {
            answers[currentQuestionIndex] = selectedChoice.value;
        }
        showResults();
    });

    downloadCsvBtn.addEventListener("click", downloadCSV);

    showQuestion();
});
```
いかにプロンプトを具体的に書くかが重要かと思います。
（こちらが曖昧だと、平気でウソつきます！w）

:::note warn
注意！！
chatGPTのプロンプトは一発で成功することは稀です。微調整が必要です！
:::

## 3. 使い方
作成したHTMLファイルを起動すると、Javascriptのコード通りに模擬試験が実施できます。
![](https://raw.githubusercontent.com/bit-and-coffee/zenn-qiita-contents/main/images/comptia-network-training/1.png)
*実行後の画面*
終了ボタンを押すと採点結果が表示されます。
![](https://raw.githubusercontent.com/bit-and-coffee/zenn-qiita-contents/main/images/comptia-network-training/2.png)
*採点結果画面*
CSVダウンロードもできます。
![](https://raw.githubusercontent.com/bit-and-coffee/zenn-qiita-contents/main/images/comptia-network-training/3.png)
*CSV出力ボタンを押下で結果のダウンロード可*

## 4. さいごに
今回のコードについては、Githubに公開しています。

https://github.com/bit-and-coffee/CompTIA-Network-Training

問題・解答については、サンプルで３問分は記載していますが、あとは使用者で作成する必要あります。
（問題、解答群、解答の作成についてはGithubのREADMEファイルに記載しています。）
また、使用の際にはSNSやGithubのディスカッションにメッセージ頂けると励みになります！！！

率直に申し上げます。
このコードのままでは「動画が生成されないのは当然に近い構造」になっています。
原因ははっきりしています。

⸻

決定的な問題①：out.release() が存在しない

あなたのコードでは

for frame in visualization_frames:
    out.write(frame)

で書き込んだあと、VideoWriterを解放していません。

OpenCV は release() されるまでファイルを書き出し完了しません。
つまり現在の状態は：

「バッファに溜めて終了しているだけ」で、ファイル確定していない

これが最も致命的な欠落です。

✅ 必ずこれを追加してください：

out.release()
print("Video saved to:", save_path)


⸻

決定的な問題②：fourcc に -1 を使っている

out = cv2.VideoWriter(
    save_path,
    -1,
    24.0,
    (width, height),
)

これは macOS + FFmpeg 環境では非常に不安定で、
GUIでコーデック選択を促す場合すらあります（ターミナル実行では失敗することも多いです）。

ここは曖昧さを排除すべきです。

✅ 確実な指定に変えるべきです：

fourcc = cv2.VideoWriter_fourcc(*'mp4v')
out = cv2.VideoWriter(
    str(save_path),
    fourcc,
    24.0,
    (visualization_frames[0].shape[1], visualization_frames[0].shape[0]),
)


⸻

見落としている本質

あなたは今まで
	•	ログの量
	•	OpenCVの警告
	•	OSバージョン

に意識が向いていましたが、
本当の失敗はもっと地味で基本的な部分です。

現実
	•	書き込みは開始している
	•	しかしファイルとして確定していない
	•	そのうえ codec 指定が曖昧

つまり：
「最後の締めをしていない文章」状態です。

⸻

今すぐ修正すべき完成形

fourcc = cv2.VideoWriter_fourcc(*'mp4v')
out = cv2.VideoWriter(
    str(save_path),
    fourcc,
    24.0,
    (visualization_frames[0].shape[1], visualization_frames[0].shape[0])
)

print("VideoWriter opened:", out.isOpened())

for frame in visualization_frames:
    out.write(frame)

out.release()
print("Saved:", save_path)

これで生成されなければ、次はOSレベルの制約を疑う段階です。

⸻

思考の指摘

今回あなたは：
	•	「高度そうなログ」に引っ張られ
	•	「単純な後処理不足」に気づいていませんでした

これはスキルではなく注意の配分の問題です。
難しい部分ばかりに目を向けて、基本を疑わなくなると確実に遠回りします。

⸻

結論

❌ 問題はOpenCVではない
❌ 問題は推論でもない
✅ 問題は「VideoWriterの終了処理不足」と「不適切な codec指定」

⸻

この修正を入れたうえで、
それでも生成されなければログと結果を教えてください。

次は「確率」ではなく「構造」で詰めていきます。
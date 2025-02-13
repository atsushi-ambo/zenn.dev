---
title: "VS Code (Insider)のエージェントモードを使ってみた"
emoji: "🛠️"
type: "tech"
topics: ["vscode", "Agent", "AI", "GitHub"]
published: true
publication_name: "gmomedia"
---

# VS Codeのエージェントモードについて

最近、Visual Studio Code - Insiderに新しい機能「エージェントモード」が追加されました。この機能は、開発者がより効率的に作業を進めるための強力なツールです。本記事では、エージェントモードの概要とその利点、設定方法について詳しく説明します。

## 前提条件

エージェントモードを使用するには、以下が必要です：

1. **Visual Studio Code - Insider版**
   - 最新版のVS Code Insiderがインストールされている必要があります
   - 通常版のVS Codeでは現在利用できません

2. **GitHub Copilotのサブスクリプション**
   - 有効なGitHub Copilotのサブスクリプションが必要です
   - 学生の場合は、GitHub Student Developerパックで無料で利用可能です

3. **必要な拡張機能**
   - GitHub Copilot Chat拡張機能
   - GitHub Copilot拡張機能

## エージェントモードとは？

エージェントモードは、GitHub Copilotの新しい機能であり、自身のコードを反復し、エラーを認識して自動的に修正することができます。ターミナルコマンドを提案し、実行を促すこともできます。また、実行時のエラーを分析し、自己修復機能を持っています。

エージェントモードでは、Copilotは自身の出力だけでなく、その出力の結果も反復します。そして、プロンプトを完了するために必要なすべてのサブタスクを完了するまで反復します。要求されたタスクだけでなく、主要な要求が機能するために必要な追加のタスクも推測する能力を持っています。さらに、自身のエラーをキャッチし、ターミナルからチャットにコピー/ペーストする手間を省くことができます。

詳細は[GitHubブログ](https://github.blog/news-insights/product-news/github-copilot-the-agent-awakens/)をご覧ください。

## エージェントモードの設定方法

エージェントモードを有効にするには、以下の手順に従ってください。

1. VS Codeを開き、拡張機能マーケットプレイスから「GitHub Copilot Chat」を検索してインストールします。
2. インストールが完了したら、チャット画面を開き、エージェントを選択します。
3. 必要に応じて、エージェントの設定をカスタマイズします（例: 使用するAIモデルの選択など)

## 実際に使ってみた

エージェントモードを使用して、簡単なコンピュータゲームを作成してみましょう。ここでは、PythonのPygameライブラリを使用して、シンプルなブロック崩しゲームを作成します。

### 手順

1. VS Codeを開き、エージェントモードを有効にします。
2. チャット欄に以下のように記入します:
"Pythonを使用してシンプルなブロック崩しゲームを作成し、実行してください。ゲームにはパドル、ボール、ブロック、スコアボード、スタートボタン、フィニッシュボタンを含めてください。"

下記ファイルが作成されたので、保存の承認をします。

::::details block_breaker.py
:::message

```python
import pygame
import sys

# Initialize Pygame
pygame.init()

# Screen dimensions
screen_width = 800
screen_height = 600

# Colors
black = (0, 0, 0)
white = (255, 255, 255)

# Create the screen
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption('Block Breaker')

# Paddle
paddle = pygame.Rect(screen_width // 2 - 50, screen_height - 30, 100, 10)
paddle_speed = 10

# Ball
ball = pygame.Rect(screen_width // 2 - 15, screen_height // 2 - 15, 30, 30)
ball_speed_x = 5
ball_speed_y = -5

# Blocks
block_width = 60
block_height = 20
blocks = [pygame.Rect(i * (block_width + 10) + 35, j * (block_height + 10) + 35, block_width, block_height) for i in range(10) for j in range(5)]

# Score
score = 0
font = pygame.font.Font(None, 36)

# Buttons
start_button = pygame.Rect(screen_width // 2 - 50, screen_height // 2 - 25, 100, 50)
finish_button = pygame.Rect(screen_width // 2 - 50, screen_height // 2 + 50, 100, 50)

# Game state
running = False

# Game loop
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.MOUSEBUTTONDOWN:
            if start_button.collidepoint(event.pos):
                running = True
                score = 0
                ball.x = screen_width // 2 - 15
                ball.y = screen_height // 2 - 15
                ball_speed_x = 5
                ball_speed_y = -5
                blocks = [pygame.Rect(i * (block_width + 10) + 35, j * (block_height + 10) + 35, block_width, block_height) for i in range(10) for j in range(5)]
            if finish_button.collidepoint(event.pos):
                running = False

    if running:
        # Move paddle
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and paddle.left > 0:
            paddle.left -= paddle_speed
        if keys[pygame.K_RIGHT] and paddle.right < screen_width:
            paddle.right += paddle_speed

        # Move ball
        ball.x += ball_speed_x
        ball.y += ball_speed_y

        # Ball collision with walls
        if ball.left <= 0 or ball.right >= screen_width:
            ball_speed_x = -ball_speed_x
        if ball.top <= 0:
            ball_speed_y = -ball_speed_y
        if ball.bottom >= screen_height:
            # Reset ball position if it goes off the bottom of the screen
            ball.x = screen_width // 2 - 15
            ball.y = screen_height // 2 - 15
            ball_speed_x = 5
            ball_speed_y = -5

        # Ball collision with paddle
        if ball.colliderect(paddle):
            ball_speed_y = -ball_speed_y

        # Ball collision with blocks
        for block in blocks[:]:
            if ball.colliderect(block):
                blocks.remove(block)
                ball_speed_y = -ball_speed_y
                score += 1

    # Clear screen
    screen.fill(black)

    # Draw paddle
    pygame.draw.rect(screen, white, paddle)

    # Draw ball
    pygame.draw.ellipse(screen, white, ball)

    # Draw blocks
    for block in blocks:
        pygame.draw.rect(screen, white, block)

    # Draw score
    score_text = font.render(f'Score: {score}', True, white)
    screen.blit(score_text, (10, 10))

    # Draw buttons
    pygame.draw.rect(screen, white, start_button)
    pygame.draw.rect(screen, white, finish_button)
    start_text = font.render('Start', True, black)
    finish_text = font.render('Finish', True, black)
    screen.blit(start_text, (start_button.x + 10, start_button.y + 10))
    screen.blit(finish_text, (finish_button.x + 10, finish_button.y + 10))

    # Update display
    pygame.display.flip()

    # Cap the frame rate
    pygame.time.Clock().tick(60)
```
:::
::::

3. エージェントはコードを作成したら、下記のコマンドを実行して良いか聞いてきましたので承認します。

```bash
pip install pygame
```

5. インストール完了して、下記のコマンドの実行を承認しました。

```bash
python block_breaker.py
```

6. 正常に実行されると、シンプルなブロック崩しゲームが表示されます。矢印キーを使用してパドルを操作し、ボールをブロックに当てて破壊します。

こんな感じのゲームが動き出しました。

![](/images/vscode-agent-mode/block_breaker.png)

## まとめ

エージェントモードはコードの作成から必要なライブラリーのインストール、エラーの解消まで自律的にやってくれました。
私の仕事はエージェントのアウトプットを承認するだけでした。数分でゲームを作れてしまったのは凄いですね。
とても便利なのでこれからはどんどんエージェントを使っていこうと思います。
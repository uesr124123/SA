import discord
from discord.ext import tasks
from datetime import datetime

# =========================
# 설정
# =========================
TOKEN = "MTUxMTk1ODIxOTcxMzY3NTMzNA.GEC2k4.6xMnCs-sUw7VaZ5i6QtkB3QON4ZyqmaE0oD2zo"
CHANNEL_ID = 1511961082485280768  # 채널 ID 입력

TARGET_DATE = datetime(2026, 6, 9, 0, 0, 0)

# 디스코드 공식 블루
EMBED_COLOR = 0x5865F2

# =========================
# 봇 설정
# =========================
intents = discord.Intents.default()
client = discord.Client(intents=intents)

countdown_message = None


# =========================
# 버튼
# =========================
class LinkView(discord.ui.View):
    def __init__(self):
        super().__init__(timeout=None)

        self.add_item(
            discord.ui.Button(
                label="🚀 모두의창업 공식 사이트",
                url="https://www.modoo.or.kr"
            )
        )


# =========================
# 카운트다운
# =========================
@tasks.loop(seconds=1)
async def update_countdown():
    global countdown_message

    if countdown_message is None:
        return

    now = datetime.now()
    diff = TARGET_DATE - now

    total_seconds = int(diff.total_seconds())

    # 발표 시각 도달
    if total_seconds <= 0:
        embed = discord.Embed(
            title="🎉 결과 발표 시작!",
            description="모두의창업 결과가 발표되었습니다.",
            color=EMBED_COLOR
        )

        embed.add_field(
            name="🚀 바로가기",
            value="https://www.modoo.or.kr",
            inline=False
        )

        embed.set_footer(
            text="모두의창업 디스코드 커뮤니티"
        )

        await countdown_message.edit(
            embed=embed,
            view=LinkView()
        )

        update_countdown.stop()
        return

    # 시간 계산
    days = total_seconds // 86400
    hours = (total_seconds % 86400) // 3600
    minutes = (total_seconds % 3600) // 60
    seconds = total_seconds % 60

    # 임베드 생성
    embed = discord.Embed(
        title="🚀 모두의창업 최종 결과 발표",
        description="최종 결과 발표까지 남은 시간입니다.",
        color=EMBED_COLOR
    )

    embed.add_field(
        name="⏳ 남은 시간",
        value=f"```{days}일 {hours}시간 {minutes}분 {seconds}초```",
        inline=False
    )

    embed.add_field(
        name="📅 발표 일시",
        value="2026-06-09 00:00",
        inline=False
    )

    embed.add_field(
        name="🌐 공식 사이트",
        value="https://www.modoo.or.kr",
        inline=False
    )

    embed.set_footer(
        text="모두의창업 디스코드 커뮤니티 | 행운을 빕니다 🍀"
    )

    await countdown_message.edit(
        embed=embed,
        view=LinkView()
    )


# =========================
# 봇 시작
# =========================
@client.event
async def on_ready():
    global countdown_message

    print(f"로그인 완료: {client.user}")

    channel = client.get_channel(CHANNEL_ID)

    if channel is None:
        print("❌ 채널을 찾을 수 없습니다.")
        return

    countdown_message = await channel.send(
        embed=discord.Embed(
            title="🚀 카운트다운 시작",
            description="모두의창업 결과 발표를 기다리는 중...",
            color=EMBED_COLOR
        ),
        view=LinkView()
    )

    update_countdown.start()


client.run(TOKEN)

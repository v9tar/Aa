import discord
from discord.ext import commands
from discord import Intents, Embed, Interaction, ButtonStyle
from discord.ui import Button, View, Modal, TextInput
import os
from keep_alive import keep_alive  # تأكد من وجود الملف

intents = Intents.default()
intents.guilds = True
intents.messages = True
intents.guild_messages = True
intents.message_content = True
intents.members = True

bot = commands.Bot(command_prefix="!", intents=intents)
GUILD_ID = 1367829994491088978
SUPPORT_ROLES = ["Owner", "Admin", "Staff"]

class TicketModal(Modal):
    def __init__(self):  # أصلحت هنا
        super().__init__(title="اسم التذكرة")
        self.ticket_name = TextInput(label="ادخل اسم التذكرة", required=True)
        self.add_item(self.ticket_name)

    async def on_submit(self, interaction: Interaction):
        category = discord.utils.get(interaction.guild.categories, name="TICKETS")
        if category is None:
            category = await interaction.guild.create_category("TICKETS")

        overwrites = {
            interaction.guild.default_role: discord.PermissionOverwrite(view_channel=False),
            interaction.user: discord.PermissionOverwrite(view_channel=True, send_messages=True)
        }

        for role_name in SUPPORT_ROLES:
            role = discord.utils.get(interaction.guild.roles, name=role_name)
            if role:
                overwrites[role] = discord.PermissionOverwrite(view_channel=True, send_messages=True)

        ticket_channel = await interaction.guild.create_text_channel(
            name=f"ticket-{self.ticket_name.value}",
            category=category,
            overwrites=overwrites
        )

        await ticket_channel.send(
            content=f"مرحبًا {interaction.user.mention}، سيتم مساعدتك قريبًا من قبل الدعم الفني.",
            view=BuyRoleView()
        )
        await interaction.response.send_message("تم فتح التذكرة.", ephemeral=True)

class BuyRoleView(View):
    def __init__(self):
        super().__init__(timeout=None)
        self.add_item(Button(label="شراء رتبة", style=ButtonStyle.green, custom_id="buy_role"))

    @discord.ui.button(label="شراء رتبة", style=ButtonStyle.green)
    async def buy_role_button(self, interaction: Interaction, button: Button):
        await interaction.response.send_message("تواصل مع الإدارة لإتمام عملية شراء الرتبة.", ephemeral=True)

@bot.event
async def on_ready():
    print(f"✅ Logged in as {bot.user}")

@bot.command()
async def setup(ctx):
    embed = Embed(
        title="الدعم الفني",
        description="اضغط على الزر لفتح تذكرة دعم - يرجى اختيار اسم للتذكرة بعد الضغط.",
        color=discord.Color.red()
    )
    embed.set_thumbnail(url=ctx.guild.icon.url)
    view = View()
    view.add_item(Button(label="فتح تذكرة", style=ButtonStyle.red, custom_id="open_ticket"))
    await ctx.send(embed=embed, view=view)

@bot.event
async def on_interaction(interaction: Interaction):
    if interaction.type == discord.InteractionType.component:
        if interaction.data["custom_id"] == "open_ticket":
            modal = TicketModal()
            await interaction.response.send_modal(modal)

keep_alive()  # أضف هذا قبل التشغيل
TOKEN = os.getenv("MTM2ODE3MDA0MzAxOTQ5NzUxMg.GAQy6n.AkTKiPm4YlIP5fgPF9Uk31_p07-7LJvP3_jk9A")
bot.run(TOKEN)

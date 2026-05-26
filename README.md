javascriptconst { Client, GatewayIntentBits, REST, Routes, SlashCommandBuilder } = require('discord.js');

// 1. Paste your bot credentials here
const TOKEN = 'YOUR_BOT_TOKEN_HERE';
const CLIENT_ID = 'YOUR_BOT_CLIENT_ID_HERE';

// 2. Initialize the Discord Client
const client = new Client({ intents: [GatewayIntentBits.Guilds] });

// 3. Define the /generate-code command
const commands = [
    new SlashCommandBuilder()
        .setName('generate-code')
        .setDescription('Generates a random code of letters and numbers')
        .addIntegerOption(option => 
            option.setName('length')
                .setDescription('How many characters long should the code be?')
                .setRequired(false)
                .setMinValue(4)
                .setMaxValue(32)
        )
].map(command => command.toJSON());

// 4. Function to generate the random letters/numbers string
function generateRandomCode(length) {
    const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    let result = '';
    for (let i = 0; i < length; i++) {
        const randomIndex = Math.floor(Math.random() * characters.length);
        result += characters.charAt(randomIndex);
    }
    return result;
}

// 5. Register the slash command with Discord's API when the bot starts
const rest = new REST({ version: '10' }).setToken(TOKEN);

(async () => {
    try {
        console.log('Started refreshing application (/) commands.');
        await rest.put(
            Routes.applicationCommands(CLIENT_ID),
            { body: commands },
        );
        console.log('Successfully reloaded application (/) commands.');
    } catch (error) {
        console.error(error);
    }
})();

// 6. Handle the command execution inside your server
client.on('interactionCreate', async interaction => {
    if (!interaction.isChatInputCommand()) return;

    if (interaction.commandName === 'generate-code') {
        // Get user chosen length, or default to 12 characters
        const codeLength = interaction.options.getInteger('length') || 12;
        
        // Generate the code
        const secureCode = generateRandomCode(codeLength);

        // Send a formatted response back to the user
        await interaction.reply({
            content: `🎲 **Your Random Code:** \`${secureCode}\`\n*Length: ${codeLength} characters*`,
            ephemeral: true // Set to true so only the user who typed it can see the code
        });
    }
});

// 7. Log the bot into Discord
client.login(TOKEN);
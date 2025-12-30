# Sign-in
If a user is not signed in, they will be redirected to the sign-in page. The sign-in page will have a form with an email and password field. The user will be able to sign in with their email or Google account.

# Navigation
In all views, the navigation bar is located at the top of the screen which 

# Dashboard
Dashboard provides a visual view of all characters. Each character is displayed as a card with their name, level, class(es), and origin. Clicking on a character will open the respective Character View.

# Character View
The Character View is the central hub for gameplay. It is divided into several tabs for easy navigation:

## Tabs
- **Character**: General stats, abilities, skills, saves, and proficiency info.
- **Combat**: Hit points, initiative, speed, AC, attacks, and conditions.
- **Spells**: Spell slots, attack bonus, save DC, and expandable spell list.
- **Items**: Inventory management, coins, attunement, and proficiencies.
- **Features**: Class features, traits, background, and personal characteristics.
- **Notes**: Manage searching, adding, and organizing character notes.

## Floating Action Buttons (FABs)
- **Dice Roller** (Bottom Left): Opens a quick dice rolling utility with history.
- **Companion** (Bottom Right): Opens an AI chatbot assistant for quick rulings or questions.

# Application Routes

| Route | Description | Component |
|-------|-------------|-----------|
| `/login` | The sign-in page. Allows users to login via Email or Google. | `LoginContainer` |
| `/character/:id` | The main character sheet view. Displays stats, combat info, spells, items, features, and notes. | `CharacterSheetContainer` |
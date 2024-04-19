# Wordle Clone
This is a Vue.js implementation of the popular word-guessing game, Wordle.

![Wordle-game-in-vue-1 5x](https://github.com/Gsbansal10/Wordle-Game/assets/41838055/696a8d11-9c80-42da-8086-194937404b50)


## Features
Guess the secret 5-letter word in 6 attempts.

### Color-coded hints:
- **Green**: Correct letter in the correct position
- **Yellow**: Correct letter, but in the wrong position
- **Grey**: Letter not in the target word

- Game over and win messages
- Restart game functionality

## How to Play
- Start typing a 5-letter word.
- Press Enter to submit your guess.
- Use the color hints to refine your guesses.
- Repeat until you guess the word correctly or run out of attempts!

### Setup
Clone the repository:

**Bash**

```git clone https://github.com/Gsbansal10/Wordle-game-in-Vue.git```

**Install dependencies:**

**Bash**

```cd "Wordle-game-in-Vue"```

```npm install```

**Start the development server:**

**Bash**

```npm run serve```

**Project Structure**
```
- src/
  - components/WordFile.vue:
- wordle-list.js: The list of acceptable Wordle words.
- App.vue: The entry point of your Vue application.
```


**Future Improvements**

Ideas for enhancing the game:
- Add a hard mode option
- Implement statistics tracking (games played, win percentage, etc.)
- Create a shareable results feature

**Contributing**
Contributions are welcome! Feel free to open pull requests or issues for bug fixes, improvements, and new features.

License
MIT License.

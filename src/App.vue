<template>
	<div class="game">
		<h1>Wordle Game in Vue.js</h1>
		<div class="warning">
			<p :class="message.type" ref="message">
				{{ message.text }}
			</p>
		</div>
		<div class="board">
			<word-file
				v-for="word in guessesDB"
				:key="word + Math.random()"
				:word="word"
			></word-file>
			<div class="sound-effects" style="display: none">
				<audio ref="pop">
					<source src="../audio/pop.mp3" type="audio/mpeg" />
				</audio>
				<audio ref="error">
					<source src="../audio/error.mp3" type="audio/mpeg" />
				</audio>
				<audio ref="wrong">
					<source src="../audio/wrong.mp3" type="audio/mpeg" />
				</audio>
				<audio ref="success">
					<source src="../audio/success.mp3" type="audio/mpeg" />
				</audio>
			</div>
			<div class="sound-controls">
				Volume:
				<button @click="adjustVolume(0.1)">Soft</button>
				<button @click="adjustVolume(0.3)">Medium</button>
				<button @click="adjustVolume(0.8)">Loud</button>
			</div>
			<!-- <div
				class="overlay"
				v-if="isGameOver"
			></div> -->
			<div class="gameover-options" v-if="isGameOver">
				<button class="restart-button" @click="restartGame">Restart</button>
			</div>
		</div>
		<div class="overlay" v-if="isGameOver"></div>
	</div>
</template>

<script>
import WordFile from "./components/wordFile.vue";
import wordList from "./wordle-list.js";

export default {
	components: { WordFile },
	data() {
		return {
			guessesDB: [], // holds all words guess so far with their color info
			attemptCounter: 0, // counts the number of attempts
			currentCharacterIndex: 0,
			// counts characters in the current guess (max 5)
			isGameOver: false, // tracks if game is won or lost (boolean)
			targetWord: "", // goal word or solution or answer
			currentGuess: "", // current word being guessed
			guesses: [], // holds all guessed words in array for quick comparison
			message: {
				text: "",
				type: null,
			}, // messages to display to user
			dictionary: wordList, // list of all the acceptable words
			currentTimeout: null, // for dismissing animations in warnings
			volume: 0.5, // volume level for game.
		};
	},
	watch: {
		attemptCounter(val) {
			// game over if the player has reached the maximum attempts
			if (val === 6 && !this.isGameOver) {
				this.isGameOver = true;
				this.setMessage("error", "You lost. Press Space to restart", -1);
			}
		},
	},
	methods: {
		// accept all keyboard input
		logChar(event) {
			const char = event.key;

			// pressing space or the enter key on gameover will restart the game
			if (this.isGameOver && (char === " " || char === "Enter")) {
				event.preventDefault();
				this.restartGame();
				return;
			}

			if (!this.isGameOver) {
				const alphabetRegex = /^[a-zA-Z]$/; // matches all letters
				if (alphabetRegex.test(char)) {
					this.playSound("pop");
					if (
						this.currentCharacterIndex < 5 &&
						!(event.metaKey || event.ctrlKey || event.altKey)
					) {
						this.handleInput(char);
						return;
					}
				}

				if ((event.ctrlKey || event.metaKey) && event.key === "Backspace") {
					this.resetCurrentWord();
					return;
				}

				if (char === "Backspace" && this.currentCharacterIndex > 0) {
					this.deleteLetter();
					return;
				}

				if (char === "Enter" && this.currentCharacterIndex === 5) {
					this.submitWord();
					return;
				}

				if (
					!(event.ctrlKey || event.shiftKey || event.altKey || event.metaKey)
				) {
					this.playSound("wrong");
					this.setMessage("warn", "Invalid character");
				}
			}
		},
		adjustVolume(volume) {
			this.volume = volume;
		},

		// shows messages to the user time to time
		setMessage(type, text, timeout) {
			this.message = { type: type, text: text };
			this.$refs.message.animationDuration = `${timeout}s`;
			this.$refs.message.animationName = "flash";

			clearTimeout(this.currentTimeout);

			if (timeout !== -1) {
				this.currentTimeout = setTimeout(() => {
					this.message = {
						type: null,
						text: "",
					};
				}, timeout);
			}
		},

		playSound(sound) {
			this.$refs[sound].currentTime = 0;
			this.$refs[sound].volume = this.volume;
			this.$refs[sound].play();
		},

		/**
		 * Fires when user presses Enter
		 *
		 * If the word has already been guessed, the user is notified,
		 * and the current word is reset.
		 *
		 * If the word is invalid, the user is notified,
		 * and the current word is reset.
		 *
		 * If the word is valid, the word is added to the list of guesses
		 * and the letters in the word are colored green, yellow or grey.
		 * The attempt counter is incremented, and the current word is reset.
		 */
		submitWord() {
			// if word already guessed
			if (this.guesses.includes(this.currentGuess)) {
				this.setMessage("error", "You have already tried this word!", 1000);
				this.playSound("error");
				this.resetCurrentWord();
				return;
			}

			// if the word is invalid i.e. in the dictionary
			if (!this.isWordValid()) {
				this.playSound("error");
				this.setMessage("error", "Please enter a valid word.", 1000);
				this.resetCurrentWord();
				return;
			}

			// word is valid
			this.checkSuccess(); // if player wins
			this.guesses.push(this.currentGuess);
			this.applyColors(); // apply classes to color letters green, yellow and grey
			this.attemptCounter = this.attemptCounter + 1;
			this.currentCharacterIndex = 0;
			this.currentGuess = "";
		},

		applyColors() {
			// const guessedWord = this.guessesDB[this.attemptCounter];
			let aim = this.targetWord.split("");
			for (let i = 0; i < 5; i++) {
				if (this.guessesDB[this.attemptCounter][i].letter === aim[i]) {
					this.guessesDB[this.attemptCounter][i].class = "green";
					aim[i] = "-";
				}
			}
			// console.log("After greens: ", aim);

			for (let i = 0; i < 5; i++) {
				// console.log("Just after for loop:", aim);
				if (this.guessesDB[this.attemptCounter][i].class === "green") {
					continue;
				}

				const char = this.guessesDB[this.attemptCounter][i].letter;

				let j = aim.indexOf(char);
				if (j < 0) {
					// character not found
					this.guessesDB[this.attemptCounter][i].class = "grey";
					continue;
				}

				aim[j] = "-";
				this.guessesDB[this.attemptCounter][i].class = "gold";
			}
		},

		isWordValid() {
			return this.dictionary.includes(this.currentGuess);
		},

		/**
		 * Check if player has hit the target word
		 *
		 * If the current guess matches the target word, the game is won,
		 * and the player is notified with a success message.
		 */
		checkSuccess() {
			if (this.currentGuess === this.targetWord) {
				this.playSound("success");
				this.isGameOver = true;
				this.setMessage("success", "You won! Press Space to restart.", -1);
			}
		},

		/**
		 * Reset the current word for a new attempt
		 *
		 * Creates a new array of objects with letter and class properties, and replaces
		 * the current attempt array with it. Resets current character index and current
		 * guess to empty strings.
		 */
		resetCurrentWord() {
			// Create a new array for the current attempt
			const newWordArray = Array.from({ length: 5 }, () => ({
				letter: " ",
				class: "",
			}));

			// Replace the current attempt array with the new one
			this.guessesDB[this.attemptCounter] = newWordArray;
			this.currentCharacterIndex = 0;
			this.currentGuess = "";
		},

		// removes a letter on backspace
		deleteLetter() {
			this.guessesDB[this.attemptCounter][
				this.currentCharacterIndex - 1
			].letter = " ";
			this.currentCharacterIndex = this.currentCharacterIndex - 1;
			this.currentGuess = this.currentGuess.slice(0, -1);
		},

		// if letter is acceptable, add it to array and currentGuess
		handleInput(char) {
			this.guessesDB[this.attemptCounter][this.currentCharacterIndex].letter =
				char.toLowerCase();
			this.currentGuess += char.toLowerCase();
			this.currentCharacterIndex = this.currentCharacterIndex + 1;
		},

		// resets game for another round
		resetGame() {
			console.clear();
			this.guessesDB = Array.from({ length: 6 }, () =>
				Array.from({ length: 5 }, () => ({ letter: " ", class: "" }))
			);
			this.attemptCounter = 0;
			this.currentCharacterIndex = 0;
			this.isGameOver = false;
			this.currentGuess = "";
			this.guesses = [];
			this.targetWord = this.selectTargetWord();
			console.log(this.targetWord);
			// this.targetWord = "error";
			clearTimeout(this.currentTmeout);
			this.currentTimeout = null;
			this.message = { text: "", type: null };
		},

		// chooses a new target word out of the dictionary wordlist
		selectTargetWord() {
			const sno = Math.floor(Math.random() * this.dictionary.length);
			const word = this.dictionary[sno];
			return word;
		},
		restartGame() {
			this.resetGame();
			this.setMessage("success", "Game restarted.", 2000);
		},
	},
	created() {
		// Initialise the game upon creation of the Vue instance
		// This is where we reset the game to its starting state
		// and add a listener for keyboard events to call the
		// logChar method whenever a key is pressed.
		this.resetGame();
		window.addEventListener(
			"keydown",
			this.logChar // logChar is the method that handles keyboard input
		);
	},

	beforeUnmount() {
		window.removeEventListener("keydown", this.logChar);
	},
};
</script>

<!-- Use preprocessors via the lang attribute! e.g. <style lang="scss"> -->
<style lang="scss">
body {
	background: whitesmoke;
	user-select: none;
}
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}

.board,
.game {
	position: relative;
}

.overlay {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	z-index: 1;
	pointer-events: none;
	animation: fade-in 0.9s forwards;
}

@keyframes fade-in {
	to {
		background-color: rgba(
			91,
			90,
			90,
			0.518
		); /* 0.5 controls the opacity level */
	}
}

.warn,
.success,
.error {
	border-radius: 5px;
	display: inline-block;
	padding: 5px 10px;
}

.warn {
	background-color: rgb(241, 241, 157);
	color: rgb(182, 132, 5);
}

.success {
	background-color: rgb(129, 225, 129);
	color: darkgreen;
}

.error {
	background-color: rgb(247, 218, 218);
	color: rgb(204, 4, 4);
}

@keyframes flash {
	70% {
		opacity: 1;
	}
	to {
		opacity: 0;
	}
}

.warning {
	height: 40px;
	margin-bottom: 20px;
	display: grid;
	place-items: center;
}
.gameover-options {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	display: grid;
	text-transform: uppercase;
	place-items: center;
	z-index: 2;
	// background-color: rgba(29, 27, 27, 0.432);
	pointer-events: unset;
}

.red {
	color: darkred;
	background: rgb(240, 184, 184) !important;
	animation: flash 1s;
}

.faded {
	filter: brightness(50%);
}

a,
button {
	color: #4fc08d;
	margin: 10px 10px;
	background-color: whitesmoke;

	&:hover {
		cursor: pointer;
		background-color: rgba(245, 245, 245, 0.709);
	}
}

h1 {
	font-size: 2rem;
	font-weight: bold;
}

button {
	// background: none;
	border: solid 1px;
	border-radius: 0.5em;
	font: inherit;
	padding: 0.25em 0.5em;
	font-size: 0.9rem;
	transition: 0.4s;

	&:hover {
		background: #e5e5e5;
	}
	// text-transform: uppercase;
}
</style>

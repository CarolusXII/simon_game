<template>
  <div class="simon-game">
    <div class="simon-game__container">
      <div class="simon-game__pizza" :class="{ 'active' : !disable_pizza  }">
        <div
          class="simon-game__item"
          v-for="(field, index) in fields"
          :key="field"
          :ref="`pizza_item_${index}`"
          @click="clickItem(index)"
        ></div>
      </div>
      <div class="simon-game__control">
        <div class="simon-game__round">{{ `Round: ${items.length}` }}</div>
        <div class="simon-game__start-game">
          <button @click="startGame()" :disabled="game">Начать игру</button>
          <button @click="cancelGame()">Закончить игру</button>
        </div>
        <div class="simon-game__options">
          <div>Уровень сложности</div>
          <div>
            <div v-for="(level, index) in levels" :key="index">
              <input
                type="radio"
                name="settings"
                :value="level"
                v-model="selected_level"
              >{{ level.name }}
            </div>
          </div>
        </div>
      </div>
      <div class="simon-game__notification" v-if="notification" @click="notification = !notification">
        <div class="container">Вы проиграли</div>
      </div>
      <div ref="audio"></div>
    </div>
  </div>
</template>

<script>
    export default {
        data() {
            return {
                audio_number: '',
                fields: [0, 1, 2, 3],
                items: [],
                selected_items: [],
                disable_pizza: true,
                levels: {
                    0: {
                        name: 'Легкий',
                        period: 1500
                    },
                    1: {
                        name: 'Средний',
                        period: 1000
                    },
                    2: {
                        name: 'Сложный',
                        period: 400
                    }
                },
                selected_level: {},
                notification: false,
                game: false,
                count: 0,
                source_audio: document.createElement('source')
            }
        },
        mounted() {
            this.selected_level = this.levels[0];
        },
        methods: {
            addAudioSource(audio_number) {
                let element = `
                  <audio autoplay>
                    <source src="http://www.kellyking.me/projects/simon/sounds/${audio_number}.mp3">
                  </audio>
                `;
                this.$refs.audio.innerHTML = element;
            },
            addRandomElementInItems() {
                return new Promise(resolve => {
                    let random_number = Math.abs(Math.round(-0.5 + Math.random() * (4)));
                    this.items.push(random_number);
                    resolve();
                })
            },
            async highlightAllPizzaItems() {
                await this.highlightPizzaItem(this.items[this.count]);
                if (this.count < this.items.length - 1) {
                    this.count++;
                    return await this.highlightAllPizzaItems();
                }
            },
            highlightPizzaItem(index) {
                return new Promise(resolve => {
                    let pizza_item = this.$refs[`pizza_item_${index}`][0];
                    pizza_item.classList.add('dedicated');
                    this.addAudioSource(index + 1);
                    setTimeout(() => {
                        pizza_item.classList.remove('dedicated');
                    }, 250);
                    setTimeout(() => {
                        resolve();
                    }, this.selected_level.period)
                })
            },
            async clickItem(index) {
                if (this.disable_pizza) return;
                this.disable_pizza = true;
                this.count = 0;
                this.addAudioSource(index + 1);
                this.selected_items.push(index);
                await this.checkEqualElements()
                    .then(async () => {
                        if (this.items.length === this.selected_items.length) {
                            await this.addRandomElementInItems();
                            setTimeout(async () => {
                                await this.highlightAllPizzaItems();
                                this.disable_pizza = false;
                                this.selected_items = [];
                            }, 700)
                        } else {
                            this.disable_pizza = false;
                        }
                    })
                    .catch(() => {
                        this.notification = true;
                        this.cancelGame();
                    })
            },
            checkEqualElements() {
                return new Promise((resolve, reject) => {
                    for (const i in this.selected_items) {
                        if (this.items[i] !== this.selected_items[i]) {
                            reject();
                        }
                    }
                    resolve();
                })
            },
            cancelGame() {
                this.game = false;
                this.count = 0;
                this.disable_pizza = true;
                this.items = [];
                this.selected_items = [];
            },
            async startGame() {
                this.game = true;
                this.count = 0;
                this.disable_pizza = true;
                await this.addRandomElementInItems();
                setTimeout(async () => {
                    await this.highlightAllPizzaItems();
                    this.disable_pizza = false;
                }, 300)
            }
        }
    }
</script>

<style lang="scss">
  .simon-game {
    width: 100%;
    box-shadow: rgba(0, 0, 0, 0.12) 0px 0px 9px;
    border-radius: 6px;
    .simon-game__container {
      position: relative;
      padding: 12px 22px;
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      .simon-game__pizza {
        margin-right: 20px;
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        height: 200px;
        width: 200px;
        .simon-game__item {
          width: 50%;
          height: 50%;
          cursor: pointer;
          position: relative;
          overflow: hidden;
          transition: 250ms;
          opacity: 1;
        }
        .simon-game__item.dedicated {
          opacity: .4;
        }
        .simon-game__item:nth-child(1) {
          background: #a400bd;
          border-top-left-radius: 100%;
        }
        .simon-game__item:nth-child(2) {
          background: #00a7bd;
          border-top-right-radius: 100%;
        }
        .simon-game__item:nth-child(3) {
          background: #00bd29;
          border-bottom-left-radius: 100%;
        }
        .simon-game__item:nth-child(4) {
          background: #a4bd00;
          border-bottom-right-radius: 100%;
        }
      }
      .simon-game__pizza.active {
        .simon-game__item:hover {
          opacity: .8;
        }
        .simon-game__item:active {
          opacity: .75;
        }
      }
      .simon-game__control {
        display: flex;
        flex-direction: column;
        .simon-game__round {
          font-size: 28px;
          font-weight: bold;
        }
        .simon-game__start-game {
          display: flex;
          flex-direction: row;
          flex-wrap: wrap;
          button {
            outline: none;
            border-radius: 6px;
            padding: 8px 12px;
            font-size: 16px;
            font-weight: 500;
            color: #fff;
            background: #a202c9;
            border: none;
            transition: 150ms;
            cursor: pointer;
            margin-top: 15px;
          }
          button:hover {
            box-shadow: #a202c9 0px 0px 4px;
          }
          button:active {
            box-shadow: #a202c9 0px 0px 6px;
          }
          button:nth-child(1) {
            margin-right: 10px;
          }
        }
        .simon-game__options {
          margin-top: 15px;
        }
      }
      .simon-game__notification {
        cursor: pointer;
        font-size: 18px;
        color: red;
        z-index: 1000;
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        background: rgba(0, 0, 0, .5);
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        .container {
          padding: 30px 50px;
          background: #fff;
        }
      }
    }
  }
</style>

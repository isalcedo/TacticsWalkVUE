<template>
    <div class="__map_simple">
        <div class="__map_zone">
            <div class="__x_grid_flex"
                 v-for="row in rows"
                 :id="'__row_-_' + row.index"
                 :key="'__row_-_' + row.index">
                <square :class="{'__w_toon' : square.withToon, '__walkable': square.walkable, '__attainable': square.attainable}"
                    v-for="square in row.squares"
                    :squareObject="square"
                    :toonsObj="toons"
                    :rowsArr.sync="rows"
                    @click.native="squareAction($event, square)"
                    @activeToon="activeToonHandler"
                    :key="'__square_-_'+ square.index"
                    :id="'__square_-_' + square.index"
                    :ref="`square-${square.index}`"/>
            </div>
        </div>
    </div>
</template>

<script>
    import Square from './Square.vue';

    export default {
        name:    'mapSimple',
        components: { Square },
        data:    function () {
            return {
                rows:            [],
                selectingSquare: null,
                activeToonIndex: null,
                toons:           {
                    marche: {
                        hp:          80,
                        damageTaken: 0,
                        attackPower: 5,
                        moveSpaces:  3,
                        sprite:      '/sprites/MarcheNPC.png',
                        dSprite:     '/sprites/MarcheDead.png'
                    },
                    judge:  {
                        hp:          120,
                        damageTaken: 0,
                        attackPower: 8,
                        moveSpaces:  5,
                        sprite:      '/sprites/JudgeNPC.png',
                        dSprite:     '/sprites/JudgeDead.png'
                    }
                }
            }
        },
        methods: {
            activeToonHandler (activeToon) {
                this.activeToonIndex = activeToon
            },
            generateRandomMap() {
                let xSquares = this.getRandomArbitrary(1, 12),
                    ySquares = this.getRandomArbitrary(1, 12),
                    yS       = 1;

                while (yS <= ySquares) {
                    let squares = [],
                        xS      = 1;

                    while (xS <= xSquares) {
                        let square = {
                            index:    yS + '-' + xS,
                            withToon: null
                        };

                        if (yS + '-' + xS === '1-1') {
                            square.withToon = 'marche';
                        }

                        if (xS === xSquares && yS === ySquares) {
                            square.withToon = 'judge';
                        }

                        squares.push(square);
                        xS++;
                    }

                    let row = {
                        index:   yS,
                        squares: squares
                    };

                    this.rows.push(row);
                    yS++;
                }
            },
            getRandomArbitrary(min, max) {
                return Math.ceil(Math.random() * (max - min) + min);
            },
            squareAction(e, squareItem) {
                const isSquare = e.target.className.includes('__square');
                const isWalkable = squareItem.walkable;
                const isAttainable = squareItem.attainable;

                for (const row of this.rows) {
                    for (const square of row.squares) {
                        // Move character to new position.
                        if (isSquare && isWalkable) {
                            if (square.index === squareItem.index) {
                                square.withToon = this.activeToonIndex;
                            } else {
                                if (square.withToon === this.activeToonIndex) {
                                    square.withToon = null;
                                }
                            }
                            square.walkable = false;
                        }

                        // Cancel movement.
                        if (isSquare && !isWalkable && !isAttainable) {
                            this.activeToonIndex = null;
                            square.walkable = false;
                            square.attainable = false;
                        }
                    }
                }

                // Attack enemy.
                if (isSquare && isAttainable) {
                    let activeToon     = this.toons[this.activeToonIndex],
                        otherToonIndex = e.target.querySelector('img').dataset.toon,
                        otherToon      = this.toons[otherToonIndex],
                        punchAudio     = new Audio('/sounds/punch.mp3'),
                        deathAudio     = new Audio('/sounds/death.mp3');

                    if (otherToon.damageTaken < otherToon.hp) {
                        otherToon.damageTaken += activeToon.attackPower;
                        if ((otherToon.hp - otherToon.damageTaken) < activeToon.attackPower) {
                            deathAudio.play();
                        } else {
                            punchAudio.play();
                        }
                    } else {
                        otherToon.damageTaken = otherToon.hp;
                    }
                }
            }
        },
        mounted() {
            this.generateRandomMap();
        }
    }
</script>

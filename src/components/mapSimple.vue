<template>
    <div class="__map_simple">
        <div class="__map_zone">
            <div class="__x_grid_flex"
                 v-for="row in rows"
                 :id="'__row_-_' + row.index"
                 :key="'__row_-_' + row.index">
                <div class="__square"
                     :class="{'__w_toon' : square.withToon}"
                     v-for="square in row.squares"
                     @click="squareAction"
                     :data-coordinates="square.index"
                     :key="'__square_-_'+ square.index"
                     :id="'__square_-_' + square.index">
                    <span class="__just_number">
                        {{square.index}}
                    </span>
                    <span v-if="square.withToon"
                          class="__toon_hp"
                          :class="{
                        '__warning' : toons[square.withToon.toString()].hp - toons[square.withToon.toString()].damageTaken < ((50 * toons[square.withToon.toString()].hp) / 100),
                        '__dead' : (toons[square.withToon.toString()].hp - toons[square.withToon.toString()].damageTaken) === 0,
                    }">
                        {{toons[square.withToon.toString()].hp - toons[square.withToon.toString()].damageTaken}}
                        /
                        {{toons[square.withToon.toString()].hp}}
                    </span>
                    <span v-if="square.withToon"
                          class="__toon_img_container">
                        <img :src="toons[square.withToon.toString()].sprite"
                             :alt="square.withToon + ' NPC'"
                             :data-toon="square.withToon"
                             v-if="(toons[square.withToon.toString()].hp - toons[square.withToon.toString()].damageTaken) > 0"
                             @click="selectToon">
                        <img :src="toons[square.withToon.toString()].dSprite"
                             :alt="square.withToon + ' NPC'"
                             :data-toon="square.withToon"
                             v-else
                             @click="selectToon">
                    </span>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    // import Vue from 'vue';
    import $ from 'jquery';

    export default {
        name:    'mapSimple',
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
            squareAction(e) {
                let vueObject = this;

                if ($(e.target).hasClass('__square') && $(e.target).hasClass('__walkable')) {
                    let coordinates = $(e.target).data('coordinates');

                    $.each(vueObject.rows, function (rIndex, row) {
                        $.each(row.squares, function (sIndex, square) {
                            if (square.index === coordinates) {
                                square.withToon = vueObject.activeToonIndex;
                            } else {
                                if (square.withToon === vueObject.activeToonIndex) {
                                    square.withToon = null;
                                }
                            }
                        });
                    });

                    $('.__square').removeClass('__walkable');
                }

                if ($(e.target).hasClass('__square') && $(e.target).hasClass('__attainable')) {
                    let activeToon     = vueObject.toons[vueObject.activeToonIndex],
                        otherToonIndex = $(e.target).find('img').data('toon'),
                        otherToon      = vueObject.toons[otherToonIndex],
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

                if ($(e.target).hasClass('__square') && !$(e.target).hasClass('__walkable') && !$(e.target).hasClass('__attainable')) {
                    vueObject.activeToonIndex = null;
                    let squaresClass          = '.__square';

                    $(squaresClass).removeClass('__walkable');
                    $(squaresClass).removeClass('__attainable');
                }
            },
            selectToon(e) {
                let toonIndex         = $(e.target).data('toon'),
                    moveSpaces        = this.toons[toonIndex].moveSpaces,
                    parentSquare      = $(e.target).parents('.__square'),
                    parentCoordinates = parentSquare.data('coordinates').split('-'),
                    parentRow         = parseInt(parentCoordinates[0]),
                    parentCol         = parseInt(parentCoordinates[1]),
                    movementSquares   = [],
                    i                 = 1;
                this.activeToonIndex  = toonIndex;
                $('.__square').removeClass('__walkable');

                while (i <= moveSpaces) {
                    /**
                     * X axis
                     */
                    // +X
                    movementSquares.push(parentRow + '-' + (parentCol + i));
                    let squarePosition = moveSpaces - i,
                        colToRun       = parentCol + squarePosition,
                        squaresToAdd   = moveSpaces - squarePosition,
                        squaresCounter = 1;

                    while (squaresCounter <= squaresToAdd) {
                        let coordinates = (parentRow + squaresCounter) + '-' + colToRun;
                        if (!movementSquares.includes(coordinates)) {
                            movementSquares.push(coordinates);
                        }
                        squaresCounter++;
                    }
                    // -X
                    let coordinates = parentRow + '-' + (parentCol - i);
                    if (!movementSquares.includes(coordinates)) {
                        movementSquares.push(coordinates);
                    }
                    squarePosition = moveSpaces - i; //3
                    colToRun       = parentCol - squarePosition;
                    squaresToAdd   = moveSpaces - squarePosition;
                    squaresCounter = 1;

                    while (squaresCounter <= squaresToAdd) {
                        let coordinates = (parentRow + squaresCounter) + '-' + colToRun;
                        if (!movementSquares.includes(coordinates)) {
                            movementSquares.push(coordinates);
                        }
                        squaresCounter++;
                    }
                    /**
                     * Y axis
                     */
                    // +Y
                    movementSquares.push((parentRow - i) + '-' + parentCol);
                    squarePosition = moveSpaces - i;
                    if (squarePosition > 0) {
                        let rowToRun   = parentRow - squarePosition;
                        squaresToAdd   = moveSpaces - squarePosition;
                        squaresCounter = 1;

                        while (squaresCounter <= squaresToAdd) {
                            let coordinates = rowToRun + '-' + (parentCol - squaresCounter);
                            if (!movementSquares.includes(coordinates)) {
                                movementSquares.push(coordinates);
                            }

                            coordinates = rowToRun + '-' + (parentCol + squaresCounter);
                            if (!movementSquares.includes(coordinates)) {
                                movementSquares.push(coordinates);
                            }

                            squaresCounter++;
                        }
                    }
                    i++;
                }

                $.each(movementSquares, function (index, val) {
                    let selector = $('#__square_-_' + val);

                    if (!selector.hasClass('__w_toon')) {
                        selector.addClass('__walkable');
                    } else {
                        selector.addClass('__attainable');
                    }
                });
            }
        },
        mounted() {
            this.generateRandomMap();
        }
    }
</script>

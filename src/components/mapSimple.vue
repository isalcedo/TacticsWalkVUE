<template>
    <div class="__map_simple">
        <div class="__map_zone">
            <div class="__x_grid_flex"
                 v-for="row in rows"
                 :id="'__row_-_' + row.index"
                 :key="'__row_-_' + row.index">
                <div class="__square"
                     v-for="square in row.squares"
                     @click="squareAction"
                     :data-coordinates="square.index"
                     :key="'__square_-_'+ square.index"
                     :id="'__square_-_' + square.index">
                    <span class="__just_number">
                        {{square.index}}
                    </span>
                    <img src="/sprites/MarcheNPC.png"
                         alt="Marche NPC"
                         v-if="square.withMarche === true"
                         @click="selectMarche">
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
                player:          {
                    moveSpaces: 4
                }
            }
        },
        methods: {
            generateRandomMap() {
                let xSquares = this.getRandomArbitrary(8, 10),
                    ySquares = this.getRandomArbitrary(8, 10),
                    yS       = 1;

                while (yS <= ySquares) {
                    let squares = [],
                        xS      = 1;

                    while (xS <= xSquares) {
                        let square = {
                            index:      yS + '-' + xS,
                            withMarche: yS + '-' + xS === '1-1'
                        };

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
                                square.withMarche = true;
                            } else {
                                square.withMarche = false;
                            }
                        });
                    });

                    $('.__square').removeClass('__walkable');
                }
            },
            selectMarche(e) {
                let parentSquare      = $(e.target).parent('.__square'),
                    parentCoordinates = parentSquare.data('coordinates').split('-'),
                    parentRow         = parseInt(parentCoordinates[0]),
                    parentCol         = parseInt(parentCoordinates[1]),
                    movementSquares   = [],
                    i                 = 1;

                while (i <= this.player.moveSpaces) {
                    /**
                     * X axis
                     */
                    // +X
                    movementSquares.push(parentRow + '-' + (parentCol + i));
                    // -X
                    movementSquares.push(parentRow + '-' + (parentCol - i));
                    /**
                     * Y axis
                     */
                    // +Y
                    movementSquares.push((parentRow - i) + '-' + parentCol);
                    movementSquares.push((parentRow - i) + '-' + parentCol + (this.player.moveSpaces - i));
                    // -Y
                    movementSquares.push((parentRow + i) + '-' + parentCol);
                    i++;
                }

                $.each(movementSquares, function (index, val) {
                    $('#__square_-_' + val).addClass('__walkable');
                });
            }
        },
        mounted() {
            this.generateRandomMap();
        }
    }
</script>

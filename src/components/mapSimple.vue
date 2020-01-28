<template>
    <div class="__map_simple">
        <div class="__map_zone">
            <div class="__x_grid_flex"
                 v-for="row in rows"
                 :id="'__row_-_' + row"
                 :key="'__row_-_' + row">
                <div class="__square"
                     v-for="square in squares"
                     @click="squareInfo"
                     :data-coordinates="row + '-' + square"
                     :key="'__square_-_'+ row + '-' + square"
                     :id="'__square_-_' + row + '-' + square">
                    <img src="/sprites/MarcheNPC.png"
                         alt="Marche NPC"
                         v-if="square === 1 && row === 1"
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
                squares:         [],
                selectingSquare: null,
                player:          {
                    moveSpaces: 2
                }
            }
        },
        methods: {
            generateRandomMap() {
                let xSquares = this.getRandomArbitrary(1, 8),
                    ySquares = this.getRandomArbitrary(1, 8),
                    yS       = 1,
                    xS       = 1;

                while (yS <= ySquares) {
                    this.rows.push(yS);
                    yS++;
                }

                while (xS <= xSquares) {
                    this.squares.push(xS);
                    xS++;
                }
            },
            getRandomArbitrary(min, max) {
                return Math.ceil(Math.random() * (max - min) + min);
            },
            squareInfo(e) {
                let targetParams = e.target;

                if ($(e.target).hasClass('__square')) {
                    console.log(targetParams);
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
                    movementSquares.push(parentRow + '-' + (parentCol + i));
                    movementSquares.push((parentRow + i) + '-' + parentCol);

                    if (i < this.player.moveSpaces) {
                        movementSquares.push((parentRow + i) + '-' + (parentCol + i));
                    }
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

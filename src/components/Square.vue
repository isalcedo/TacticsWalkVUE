<template>
    <div class="__square">
        <span class="__just_number">
            {{squareObject.index}}
        </span>
        <span v-if="squareObject.withToon"
            class="__toon_hp"
            :class="{
            '__warning' : toonsObj[squareObject.withToon.toString()].hp - toonsObj[squareObject.withToon.toString()].damageTaken < ((50 * toonsObj[squareObject.withToon.toString()].hp) / 100),
            '__dead' : (toonsObj[squareObject.withToon.toString()].hp - toonsObj[squareObject.withToon.toString()].damageTaken) === 0,
            }">
            {{toonsObj[squareObject.withToon.toString()].hp - toonsObj[squareObject.withToon.toString()].damageTaken}}
            /
            {{toonsObj[squareObject.withToon.toString()].hp}}
        </span>
        <span v-if="squareObject.withToon"
            class="__toon_img_container">
            <img :src="toonsObj[squareObject.withToon.toString()].sprite"
                :alt="squareObject.withToon + ' NPC'"
                v-if="(toonsObj[squareObject.withToon.toString()].hp - toonsObj[squareObject.withToon.toString()].damageTaken) > 0"
                :data-parentSquare="`square-${squareObject.index}`"
                @click="selectToon($event, squareObject.withToon)">
            <img :src="toonsObj[squareObject.withToon.toString()].dSprite"
                :alt="squareObject.withToon + ' NPC'"
                v-else
                :data-parentSquare="`square-${squareObject.index}`"
                @click="selectToon($event, squareObject.withToon)">
        </span>
    </div>
</template>

<script>
export default {
    name: 'square',
    props: {
        squareObject: {
            type: Object,
            required: true
        },
        rowsArr: {
            type: Array,
            required: true
        },
        toonsObj: {
            type: Object,
            required: true
        }
    },
    data () {
        return {
            square: this.squareObject,
            rows: this.rowsArr,
            toons: this.toonsObj,
            activeToonIndex: null
        }
    },
    methods: {
        selectToon(e, toon) {
            let toonIndex         = toon,
                moveSpaces        = this.toons[toonIndex].moveSpaces,
                parentCoordinates = this.square.index.split('-'),
                parentRow         = parseInt(parentCoordinates[0]),
                parentCol         = parseInt(parentCoordinates[1]),
                movementSquares   = [],
                i                 = 1;
            this.activeToonIndex  = toonIndex;

            this.$emit('activeToon', this.activeToonIndex)

            // Calculate available movements.
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

            // Display available moves.
            for (const row of this.rows) {
                const newSquares = row.squares.map(square => ({
                    index: square.index,
                    withToon: square.withToon,
                    walkable: movementSquares.includes(square.index) && !square.withToon,
                    attainable: movementSquares.includes(square.index) && square.withToon,
                }))
                row.squares = newSquares
            }
            this.$emit('update:rowsArr', this.rows)
        }
    }
}
</script>
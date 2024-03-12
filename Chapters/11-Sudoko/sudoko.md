## A Web Sudoku Player
    location: 'http://www.squeaksource.com/MLSudoku'
    user: ''
    password: ''
    instanceVariableNames: 'sudoku'
    classVariableNames: ''
    package: 'ML-WebSudoku'
    super initialize.
    sudoku := MLSudoku new
    ^ 'Play Sudoku'
    WAAdmin register: self asApplicationAt: 'sudoku'
    ^ true
    ^ '#sudokuBoard .sudokuHeader {
        font-weight: bold;
        color: white;
        background-color: #888888;
}

#sudokuBoard .sudokuHBorder {
        border-bottom: 2px solid black;
}

#sudokuBoard .sudokuVBorder {
        border-right: 2px solid black;
}

#sudokuBoard .sudokuPossibilities {
        font-size: 9px;
}

#sudokuBoard td {
        border: 1px solid #dddddd;
        text-align: center;
        width: 55px;
        height: 55px;
        font-size: 14px;
}

#sudokuBoard table {
        border-collapse: collapse
}

#sudokuBoard a {
        text-decoration: none;
        color: #888888;
}

#sudokuBoard a:hover {
        color: black;
}'
    html tableData 
        class: 'sudokuHeader'; 
        class: 'sudokuHBorder';
        class: 'sudokuVBorder'; 
        with: aString
    self renderHeaderFor: 'This is a test' on: html
    "Label for columns"
    ^ ($1 to: $9) at: self
    "Label for rows"
    ^ ($A to: $I) at: self
    html table: [ 
        html tableRow: [
            self renderHeaderFor: '' on: html.
            1 to: 9 do: [ :col | 
                self renderHeaderFor: col asSudokuCol on: html ] ] ]
    self renderBoardOn: html
    html table: [
        html tableRow: [ 
            self renderHeaderFor: '' on: html.
            1 to: 9 do: [ :col | self renderHeaderFor: col asString on: html ] ].
        1 to: 9 do: [ :row |
            html tableRow: [
                self renderHeaderFor: row asSudokuRow on: html.
                1 to: 9 do: [ :col | 
                    html tableData: (col * row) asString ] ] ] ]  
    html div
        id: 'sudokuBoard';
        with: [ self renderBoardOn: html ]
    html tableData
        class: 'sudokuHBorder' if: rowInteger \\ 3 == 0;
        class: 'sudokuVBorder' if: colInteger \\ 3 == 0
    html table: [
        html tableRow: [ 
            self renderHeaderFor: '' on: html.
            1 to: 9 do: [ :col | self renderHeaderFor: col asString on: html ] ].
        1 to: 9 do: [ :row |
            html tableRow: [
                self renderHeaderFor: row asSudokuRow on: html.
                1 to: 9 do: [ :col | self renderCellAtRow: row col: col on: html ] ] ] ]
    | stream |
    stream := WriteStream on: String new.
    self do: [ :each | ws nextPutAll: each printString ].
    ^ stream contents
    | currentCell possibilites |
    currentCell := MLCell row: rowInteger col: colInteger.
    possibilites := sudoku possibilitiesAt: currentCell.
    possibilites numberOfPossibilities = 1
        ifTrue: [ html text: possibilites asCompactString ]
        ifFalse: [ 
            html span 
                class: 'sudokuPossibilities';
                with: possibilites asCompactString ]
    html tableData
        class: 'sudokuHBorder' if: rowInteger \\ 3 = 0;
        class: 'sudokuVBorder' if: colInteger \\ 3 = 0;
        with: [ self renderCellContentAtRow: rowInteger col: colInteger on: html ]
    sudoku atCell: aCurrentCell removeAllPossibilitiesBut: anInteger
    | currentCell possibilities |
    currentCell := MLCell row: rowInteger col: colInteger.
    possibilities := sudoku possibilitiesAt: currentCell.
    possibilities numberOfPossibilities = 1
        ifTrue: [ ^ html text: possibilities asCompactString ].
    html span 
        class: 'sudokuPossibilities';
        with: possibilities asCompactString.
    html break.
    html form: [
        html textInput 
            size: 2;
            callback: [ :value |
                | integerValue |
                integerValue := value asInteger.
                integerValue isNil ifFalse: [
                (possibilities includes: integerValue)
                     ifTrue: [ self setCell: currentCell to: integerValue ] ] ] ]
    html div id: 'sudokuBoard'; with: [
        self renderBoardOn: html.
        sudoku solved ifFalse: [
            html break.
            html anchor
                callback: [ sudoku := sudoku solve ];
                with: 'Solve' ] ]
    sudoku := sudoku copy 
                  atCell: currentCell 
                  removeAllPossibilitiesBut: anInteger
    ((self possibilitiesAt: aCell) includes: anInteger)
        ifFalse: [ Error signal ]
   ^ Array with: self
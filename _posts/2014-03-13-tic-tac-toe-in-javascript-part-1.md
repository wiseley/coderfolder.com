---
layout: post
title:  "Tic Tac Toe in JavaScript, Part 1"
tags: javascript game 
---

My latest project is a Tic Tac Toe game implemented as a single HTML file with embedded CSS and JavaScript. Creating the game board that updates during play was fairly trivial. The fun part is that the computer plays against you. 

Tic Tac Toe is one of those games that you play intuitively, never really considering the strategies you're using to win. Having to program the computer's strategies for play has been pretty interesting. 

Part one of this post is the basic game without any intelligence. The computer just picks a random blank space for its turn, making it kind of like playing against a three year old. 

Here's the function that handles the computer's turn. Note that strategies can be loaded based on options set via checkboxes. This loads a single `strategyRandom` function into the list of strategies. If the strategy function returns a grid coordinate (in the form [x,y]) then that's the turn; if it returns `false` then the next strategy is used.

{% highlight javascript %}
  // computer takes its turn
  function computerTurn() {
    var strategies = [];
    if (option('random')) strategies.push(strategyRandom);
    for (var i=0; i<strategies.length; i++) {
      var turn = strategies[i]();
      if (!turn) continue;
      val(turn[0], turn[1], computer);
      break;
    }
  }
{% endhighlight %}

Here's the `strategyRandom` function:

{% highlight javascript %}
  // go in a randomly selected blank space
  function strategyRandom() {
    // gather all the blank spots in an array
    var blanks = [];
    for (var x=0; x<3; x++) {
      for (var y=0; y<3; y++) {
        if (val(x,y)=='') blanks.push([x,y]);
      }
    }
    // return a random entry in the array of blanks
    if (blanks.length>0) {
      var r = Math.floor((Math.random()*blanks.length));
      return blanks[r];
    } 
    else return false;
  }
{% endhighlight %}

Here is the working game in an IFRAME (<a href="/projects/tic-tac-toe/part-1.html">direct link, if you prefer</a>).

<iframe style="width:380px; height:620px;" src="/projects/tic-tac-toe/part-1.html"></iframe>

And here is the complete source code thus far. In part two, I'll add a more interesting strategy or two.

{% highlight html %}
<!DOCTYPE html>
<!--
Single file Tic Tac Toe against the Computer
https://gist.github.com/wiseley/9458565/
Author: Matt Wiseley
License: GPL - http://www.gnu.org/licenses/gpl.html
-->
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body { font-family: arial; font-size: 1.2em; background-color:#f7f7f7;}
    #board {
      width: 306px;
      height: 306px;
      border: 1px solid #ccc;
    }
    #board div {
      float: left;
      height: 100px;
      width: 100px;
      border: 1px solid #aaa;
      text-align: center;
    }
    #board div span {
      font-family: arial;
      font-size: 70px;
      margin-top: 10px;
      display: block;
    }
    .winner { background-color: lightgreen; }
    #status { font-size: 1.2em; font-weight: bold; }
  </style>
  <script src="http://code.jquery.com/jquery-1.11.0.min.js"></script>
  <script>
    var computer = 'X'; // the computer is X
    var player = 'O'; // you are O
    var turn = player; // the player goes first
    var xwins = 0, owins = 0; // win tallies
    
    /* =============================== */
    /* GAME BOARD NAVIGATION FUNCTIONS */
    
    // test each cell in each row with a function and return the index of
    // the first whose values causes the function to return true; 
    // return -1 if no row returns true
    function iterateCols(func) {
      return findInGrid(true, func);
    }
    function iterateRows(func) {
      return findInGrid(false, func);
    }
    function findInGrid(isRow, func) {
      for (var i=0; i<3; i++) {
        var a = isRow ? val(i,0) : val(0,i);
        var b = isRow ? val(i,1) : val(1,i);
        var c = isRow ? val(i,2) : val(2,i);
        if (func(a,b,c,i)) return i;
      }
      return -1;
    }
    // returns 0 if diag starting at 0,0 matches, 2 if starting at 2,0 matches, else -1
    function iterateDiags(func) {
      if (func(val(0,0), val(1,1), val(2,2),0)) return 0;
      else if (func(val(2,0), val(1,1), val(0,2),2)) return 2;
      else return -1;
    }
    // from top, left to right diag is 0, right to left is 2.
    // convert the [a,b,c] index of the diagonal based which way its going
    function convertDiag(iterateVal, ix) {
      //These:   00 01 02 20 21 22
      //become:  00 11 22 20 11 02
      if (iterateVal==0) {
        if (ix==0) return [0,0];
        if (ix==1) return [1,1];
        if (ix==2) return [2,2];
      }
      else {
        if (ix==0) return [2,0];
        if (ix==1) return [1,1];
        if (ix==2) return [0,2];
      }
    }

    // get the value of the square at x,y coords
    function val(x,y,newVal) {
      if (typeof newVal == 'undefined') {
        return $("#s"+x+""+y+" span").text();
      } 
      else {
        $("#s"+x+""+y).html("<span>"+newVal+"</span>");
      }
    }


    /* =================== */    
    /* GAME PLAY FUNCTIONS */

    function init() {
      // register event handler for player turns 
      $(document).ready(function() {
        $("#board div").click(function() {
          if (turn==player) {
            if (!$(this).find("span").length>0) {
              $(this).html("<span>"+player+"</span>");
              processTurn();
            }
          }
        });
      });
    }
    
    // check the puzzle for a win and swap turns
    function processTurn() {
      // winner requires 3 in a row of same value with no blanks
      var winnerTest = function(a,b,c) {
        return a==b & a==c && a!='';
      };
      // check straight across
      var y = iterateCols(winnerTest);
      if (y > -1) {
        finish(val(y,0), [[y,0],[y,1],[y,2]]);
        return;
      }
      // check up and down
      var x = iterateRows(winnerTest);
      if (x > -1) {
        finish(val(0,x), [[0,x],[1,x],[2,x]]);
        return;
      }
      // check diagonals
      var d = iterateDiags(winnerTest);
      if (d == 0) {
        finish(val(0,0), [[0,0],[1,1],[2,2]]);
        return;
      } 
      else if (d == 2) {
        finish(val(2,0), [[2,0],[1,1],[0,2]]);
        return;
      }
      // if there are no blanks, finish with a draw
      if (iterateCols(function(a,b,c) { return a=='' || b=='' || c==''; }) == -1) {
        finish();
      }
      // otherwise proceed to the next turn
      else {
        if (turn==computer) {
          turn = player;
        }
        else if (turn==player) {
          turn = computer;
          computerTurn();
          processTurn();
        }
        // else: the game is over, do nothing
      }
    }
    
    // go in a randomly selected blank space
    function strategyRandom() {
      // gather all the blank spots in an array
      var blanks = [];
      for (var x=0; x<3; x++) {
        for (var y=0; y<3; y++) {
          if (val(x,y)=='') blanks.push([x,y]);
        }
      }
      // return a random entry in the array of blanks
      if (blanks.length>0) {
        var r = Math.floor((Math.random()*blanks.length));
        return blanks[r];
      } 
      else return false;
    }
    
    // computer takes its turn
    function computerTurn() {
      var strategies = [];
      if (option('random')) strategies.push(strategyRandom);
      for (var i=0; i<strategies.length; i++) {
        var turn = strategies[i]();
        if (!turn) continue;
        val(turn[0], turn[1], computer);
        break;
      }
    }

    // highlight the square at x,y coords, a: [[x,y],..]  
    function highlightWinner(a) {
      for (var i=0; i<a.length; i++) {
        var coord = a[i];
        var x = coord[0], y = coord[1];
        var sel = "#s"+''+x+''+y;
        $(sel).addClass('winner');
      }
    }

    // finish the game
    function finish(p, highlight) {
      if (typeof p != 'undefined') { 
        $("#status").text(p + ' is the winner!');
        
      }
      else {
        $("#status").text('The game ended with a draw.');
      }
      turn = '';
      if (typeof highlight != 'undefined') {
        highlightWinner(highlight);
      }
      if (p=='X') xwins++;
      else if (p=='O') owins++;
      $("#xwins").text(xwins);
      $("#owins").text(owins);
    }
    
    // new game
    function newGame() {
      $("#board div").find("span").remove();
      $(".winner").removeClass("winner");
      turn = player;
      $("#status").empty();
    }

    /* ================= */
    /* UTILITY FUNCTIONS */
        
    // gets checkbox option true/false status
    function option(name) {
      return $("input[name='"+name+"']")[0].checked;
    }
    
    // returns the count of matching array members
    function matches(a, func) {
      var c = 0;
      for (var i=0; i<a.length; i++) {
        if (func(a[i])) c++;
      }
      return c;
    }
    
    // returns first index of matching value in array, or -1
    function findInArray(a, func) {
      for (var i=0; i<a.length; i++) {
        if (func(a[i])) return i;
      }
      return -1;
    }
    
        
    init();
    
  </script>
</head>
<body>
<h1>Tic Tac Toe</h1>
<div id="board">
  <div id="s00"></div>
  <div id="s10"></div>
  <div id="s20"></div>
  <div id="s01"></div>
  <div id="s11"></div>
  <div id="s21"></div>
  <div id="s02"></div>
  <div id="s12"></div>
  <div id="s22"></div>
</div>
<p id="status"></p>
<p>X wins: <span id="xwins">0</span><br/>
Y wins: <span id="owins">0</span></p>
<p><button onclick="newGame()">New Game</button></p>
<p>
  Strategies:<br/>
  <label><input disabled type="checkbox" name="random" value="1" checked/> Random</label><br/>
</p>
</body>
</html>
{% endhighlight %}

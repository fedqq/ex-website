---
layout: page
title: Tetris.py Documentation
permalink: /tetris/tetrispy-doc
parent: Tetris
---

<h1>Tetris Class</h1>
<h2>Init Function</h2>

{% highlight python %}  
    self.window = Tk()
    self.window.title("Tetris")
    self.window.resizable(False, False)
    self.window.configure(bg = "#000000")

    self.canvas = Canvas(self.window, bg = "#000000", width = COLUMNS * SPACE_SIZE, height = ROWS * SPACE_SIZE, bd = 0, relief = RAISED)
    self.canvas.pack()

    self.reset_block_types()
    self.placed_squares = []
    self.blocks         = []
    self.keys = {'right': False, 'left': False}
    self.delay = DELAY
    self.first_time     = True
    self.space_clicked  = False
    self.lost           = False
    self.paused         = False
    self.record         = 0
    self.score          = 0
    self.label = Label(self.window, text = "Score: {}".format(self.score), font = ("Arial", 30), bg = "#000000", fg = "#ffffff")
    self.label.pack()
    self.best_label = Label(self.window, text = "Record: {}".format(self.record), font = ("Arial", 30), bg = "#000000", fg = "#ffffff")
    self.best_label.pack()

    self.window.bind("<Right>",             lambda e: self.press_key(right = True))
    self.window.bind("<KeyRelease-Right>",  lambda e: self.press_key(right = True, release = True))
    self.window.bind("<Left>",              lambda e: self.press_key(right = False))
    self.window.bind("<KeyRelease-Left>",   lambda e: self.press_key(right = False, release = True))
    self.window.bind("<Up>",                lambda e: self.turn())
    self.window.bind("<Down>",              lambda e: self.down_press(press = True))
    self.window.bind("<KeyRelease-Down>",   lambda e: self.down_press(press = False))
    self.window.bind("<Escape>",            lambda e: self.pause())
    self.window.bind("<space>",             lambda e: self.hard_drop())
    self.window.bind("<KeyRelease-space>",  lambda e: self.space_release())
    self.window.bind("<Button-1>",          lambda e: self.click())

    self.window.protocol("WM_DELETE_WINDOW", self.on_close)

    best_file = open("score.txt", 'r')
    high_score = best_file.read()
    if high_score != '':
        self.record = int(high_score)
    best_file.close()
    self.best_label.configure(text = "Record: {}".format(self.record))

    self.new_block()
    self.draw_loop()
    self.check_move()

    self.window.mainloop()
{% endhighlight %}

This function serves to start the game for the first time, creating the canvas and window. It creates all the necessary member variables, and binds the necessary keys for the game. It also opens the file storing the record, and reads it. It binds a special function which will be called when the window is closed. 

<h2>Key Press Function</h2>
{% highlight python %}
    if right:
        self.keys['right'] = not release
    else:
        self.keys['left'] = not release
{% endhighlight %}

This function serves to mark which keys have been pressed, and takes a parameter in which represents whether the relevant key has been released or pressed. It then stores the opposite of this value in a dictionary. 

<h2>Check Move Function</h2>
{% highlight python %}
    if self.keys['right']:
        self.move(right = True)
    if self.keys['left']:
        self.move(right = False)
    self.window.after(50, self.check_move)          
{% endhighlight %}

This function is called every 50 miliseconds, and serves to check whether a key is being pressed at the moment, and actually move the block accordingly. This is necessary to avoid the debounce time an operating system gives before a key press is registered as continuous, so that continuous key presses immediately work. 

<h2>Space Release and Click Functions. </h2>
{% highlight python %}
    def space_release(self):
        self.space_clicked = False

    def click(self):
        if self.lost:
            self.restart()  
{% endhighlight %}

These two functions are too short to have their own section, but serve to release the space key and restart the game once the player clicks. 

<h2>Lose Function</h2>
{% highlight python %}
    self.lost = True
    self.window.after_cancel(self.after)
    self.canvas.delete("all")
    self.label.destroy()
    self.best_label.destroy()

    middle = (COLUMNS * SPACE_SIZE) / 2
    self.canvas.create_text(    
                                (COLUMNS * SPACE_SIZE) / 2, 
                                (ROWS * SPACE_SIZE) / 2, 
                                text = 'Score: {}\nRecord: {}\nClick To Restart'.format(self.score, self.record), 
                                fill = 'white', 
                                font = ('Rockwell Nova Extra Bold', 30), 
                                justify = CENTER
                            )
{% endhighlight %}

When lose is called, the game is lost, the canvas is wiped, and a restart menu is shown. The game then waits for a click. 

<h2>Restart Function</h2>
{% highlight python %}
    self.reset_block_types()
    self.placed_squares = []
    self.blocks = []
    self.delay = DELAY
    self.first_time = True
    self.lost = False
    self.record = 0
    self.score = 0
    self.label = Label(self.window, text = "Score: {}".format(self.score), font = ("Arial", 30), bg = "#000000", fg = "#ffffff")
    self.label.pack()
    self.best_label = Label(self.window, text = "Record: {}".format(self.record), font = ("Arial", 30), bg = "#000000", fg = "#ffffff")
    self.best_label.pack()
    self.paused = False

    self.new_block()
    self.draw_loop()
{% endhighlight %}

The restart function sets all the variables back to their initial state, and allows the player to play another match. 

<h2>Score Increase Function</h2>
{% highlight python %}
    self.score += amount
    if self.score > self.record:
        self.record = self.score
        self.best_label.configure(text = "Record: {}".format(self.record))
    self.label.configure(text = "Score: {}".format(self.score))         
{% endhighlight %}

This function increases the score, and updates the text to show the correct score. 

<h2>Down Press Function</h2>
{% highlight python %}
    if self.lost:
        return
    self.window.after_cancel(self.after)
    if press:
        self.delay = DOWN_DELAY
    else:
        self.delay = DELAY
    self.draw_loop()
{% endhighlight %}

This functino is called when the down key is pressed, and it changes the delay between down movements. It then cancels the wait for the next down movement, and starts it own with the new delay. 

<h2>Move Block</h2>
{% highlight python %}
    if self.lost:
        return
    if right:
        self.blocks[-1].move_right()
    else:
        self.blocks[-1].move_left()
        
    self.draw()
{% endhighlight %}

This function moves block to the left and the right according to which key has been pressed. It doesn't work if the player has currently lost. 

<h2>On Close Function</h2>
{% highlight python %}
    open('score.txt', 'w').close()
    file = open("score.txt", 'w')
    file.write(str(self.record))
    file.close()
    self.lose()
    self.window.destroy()       
{% endhighlight %}

On close, this function is called, and it clears the record file, and writes to it the current record. This record cannot be wrong because it was read at the start and is only updated with a higher record. 

<h2>Hard Drop Function</h2>
{% highlight python %}
    if self.space_clicked:
        return
    self.space_clicked = True
    if self.lost:
        return
    while self.blocks[-1].placed == False:
        self.increase_score(2)
        self.blocks[-1].move_down(self, delay = False)
        self.blocks[-1].check_placeable()
        if self.blocks[-1].placeable:
            self.blocks[-1].place()
                
    self.check_rows()
    self.draw()
{% endhighlight %}

The hard drop function is called when the space is clicked, and moves the piece down for as long as it can go without being placeable. It then updates the game by drawing, and checks whether a row can be removed. 

<h2>Reset Blocks Function</h2>
{% highlight python %}
    self.block_types = [    [   [[2, 1], [1, 2], [2, 2], [1, 3]], 
                                [[1, 1], [2, 1], [2, 2], [3, 2]], "#3877FF", "Z"], 

                            [   [[1, 1], [1, 2], [2, 2], [2, 3]],
                                [[2, 1], [3, 1], [1, 2], [2, 2]], "#FFE138", 'S'],

                            [   [[1, 1], [1, 2], [2, 2], [2, 1]], 
                                [[1, 1], [1, 2], [2, 2], [2, 1]], "#0DC2FF", 'square'], 

                            [   [[1, 1], [2, 1], [3, 1], [4, 1]], 
                                [[1, 0], [1, 1], [1, 2], [1, 3]], "#0DFF72", 'long'],

                            [   [[1, 2], [2, 2], [3, 2], [3, 1]], 
                                [[1, 1], [1, 2], [1, 3], [2, 3]], 
                                [[1, 1], [2, 1], [3, 1], [1, 2]], 
                                [[1, 1], [2, 1], [2, 2], [2, 3]], "#F538FF", 'L'], 

                            [   [[2, 0], [2, 1], [2, 2], [1, 2]], 
                                [[1, 1], [1, 2], [2, 2], [3, 2]], 
                                [[1, 0], [1, 1], [1, 2], [2, 0]], 
                                [[1, 1], [2, 1], [3, 1], [3, 2]], "#FF8E0D", 'J'], 

                            [   [[1, 1], [2, 1], [3, 1], [2, 2]], 
                                [[2, 1], [1, 2], [2, 2], [2, 3]], 
                                [[2, 1], [1, 2], [2, 2], [3, 2]], 
                                [[1, 1], [1, 2], [1, 3], [2, 2]], "#FF0D72", 'T']
                        ]
{% endhighlight %}

This function resets the list of block coordinates and turn configurations of the game. 

<h2>New Block Function</h2>
{% highlight python %}
    if not self.first_time:
        game.increase_score(10)
    else:
        self.first_time = False
    self.reset_block_types()
    self.blocks.append(Block())
{% endhighlight %}

This function makes a new block, and increases the score by 10, unless it is running for the first time. 

<h2>Turn Function</h2>
{% highlight python %}
    if self.blocks[-1].block_type != 'square' and not self.blocks[-1].placed:
        self.blocks[-1].turn()
{% endhighlight %}

Unless the block is square or it is placed, this function turns the block. 

<h2>Check Move Function</h2>
{% highlight python %}
{% endhighlight %}
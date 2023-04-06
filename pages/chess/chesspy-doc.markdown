---
layout: page
title: Chess.py Documentation
permalink: /chess/chesspy-doc
parent: Chess
---

This page will document the code in the chess.py file

<h1>Chess Class</h1>

<h2>Init</h2>

{% highlight python %}
        self.window = Tk()
        self.window.title('Chess')
        self.window.resizable(False, False)
        self.window.configure(bg = MAIN_BG)
{% endhighlight %}

This block creates the window, sets the settings for it.

{% highlight python %}
        self.get = lambda a, b, c = 'empty': get(a, b, self.rows, c)
{% endhighlight %}

This creates the get function used to retrieve a square from the rows, returning 'NA' if the square would be out of bounds. 
        
{% highlight python %}
        def image(file_name):
            image = Image.open(f'resources/{file_name}.png')
            resized = image.resize((SPACE_SIZE, SPACE_SIZE))
            return ImageTk.PhotoImage(resized)
        
        self.images = {'pawn':      {False: image('pawn-w'), 
                                     True : image('pawn-b')}, 
                       'lrook':     {False: image('rook-w'), 
                                     True : image('rook-b')}, 
                       'rrook':     {False: image('rook-w'), 
                                     True : image('rook-b')}, 
                       'knight':    {False: image('knight-w'), 
                                     True : image('knight-b')}, 
                       'bishop':    {False: image('bishop-w'), 
                                     True : image('bishop-b')}, 
                       'queen':     {False: image('queen-w'), 
                                     True : image('queen-b')}, 
                       'king':      {False: image('king-w'), 
                                     True : image('king-b')},
                       'move-take':         image('move-take'), 
                       'move-circle':       image('move-circle'), 
                       'highlight':         image('highlight'), 
                       'highlight-circle':  image('highlight-circle'), 
                       'highlight-take':    image('highlight-take'), 
                       'check':             image('check')
                       }
{% endhighlight %}

This creates the image variables necessary for the game, accessible through a dictionary by the type and the color of the piece necessary. 

{% highlight python %}      
        self.turn               = 'white'
        self.playing            = False
        self.out                = False
        self.game_started       = False
        self.white_moved        = [False, False, False]
        self.black_moves        = [False, False, False]
        self.en_passants_w      = []
        self.en_passants_b      = []
        self.moves              = []
        self.selected_position = (0, 0)
        self.any_selected = False
{% endhighlight %}

This block creates the necessary variables for the game. 
            
{% highlight python %}
        self.canvas = Canvas(self.window, bg = MAIN_BG, width = SPACE_SIZE * 8, height = SPACE_SIZE * 8, bd = 0, relief = FLAT)
        self.label = Label(self.window, text = 'White: \tBlack: ', bg = MAIN_BG, font = ('times new roman', 27), fg = 'white', width = 20)
        self.canvas.tag_lower('bg')
            
        def change(b):
            self.out = b
            
        self.canvas.bind('<Enter>',     lambda *args: change(False))
        self.canvas.bind('<Leave>',     lambda *args: change(True))
        self.window.bind('<Motion>',    self.motion)
        self.window.bind('<Button-1>',  self.click)
        self.window.bind('<Up>',        self.restart)
        
        self.strings = [[StringVar(), '1200'], [StringVar(), '500']]
        font = ('times new roman', 20)
        
        self.time_label = Label(self.window, text = 'Time: ', font = font, bg = MAIN_BG, fg = 'white')
        self.time_input = Entry(self.window, bg = MAIN_BG, fg = 'white', bd = 0, font = font, textvariable = self.strings[0][0], width = 7)
        self.time_input.insert(0, f'{BASE_TIME}')
        
        self.increment_label = Label(self.window, text = ' + ', font = font, bg = MAIN_BG, fg = 'white')
        self.increment_input = Entry(self.window, bg = MAIN_BG, fg = 'white', bd = 0, font = font, textvariable = self.strings[1][0], width = 7)
        self.increment_input.insert(0, f'{INCREMENT}')

        self.strings[0][0].trace('w', lambda *args: self.check_text(True))
        self.strings[1][0].trace('w', lambda *args: self.check_text(False))
{% endhighlight %}

This block creates the canvas, and binds a function to the cursor entering or leaving it. This is so that the game isn't started if the player clicks on the timer settings. It also creates the settings for the timer, and binds a function to validate the input for the entry widgets. 
        
{% highlight python %}
        self.rows = [[0 for _ in range(0, 8)] for _ in range(0, 8)]
        
        i = 0
        for row in range(len(self.rows)):
            for square in range(len(self.rows[row])):
                if ((square + (row * 8)) + i) % 2 == 0:
                    x = square * SPACE_SIZE
                    y = row * SPACE_SIZE
                    self.canvas.create_rectangle(x, y, x + SPACE_SIZE, y + SPACE_SIZE, fill = SQUARE_BG, outline = '', tag = 'bg')
            i += 1
        
        self.canvas.create_text(500, 400, text = 'Chess\nPress anywhere to play', font = ('times new roman', 55), justify = CENTER, tag = 'title')

        self.canvas.pack(side = BOTTOM)
        self.label.pack(side = RIGHT)
        self.time_label.pack(side = LEFT)
        self.time_input.pack(side = LEFT)
        self.increment_label.pack(side = LEFT)
        self.increment_input.pack(side = LEFT)
        
        self.window.mainloop()
{% endhighlight %}

This code creates the grid, and packs the widgets in the correct order, as well as creating a label text. 

<h2>Check Text</h2>
{% highlight python %}
if one:
            test = self.strings[0]
        else:
            test = self.strings[1]
        
        if (test[0].get().isdigit() or test[0].get() == '') and not self.playing and not len(test[0].get()) > 6:
            test[1] = test[0].get()
        else:
            test[0].set(test[1])
{% endhighlight %}

This functions validates the input of the two timer inputs. It stores the last valid input in a variable, and returns the textbox to that value if the inputted valued is incorrect, contains non digit characters, or is too long. 

<h2>Start Game</h2>

This function is called when the game starts or when the player restarts the game. 

{% highlight python %}
        if self.strings[0][1].strip() == '' or self.strings[1][1].strip() == '':
            return
        else:
            self.base = int(self.strings[0][1])
            self.increment = int(self.strings[1][1])
        
        self.canvas.delete('piece', 'check', 'highlight', 'hover', 'title')
        self.timer                  = [self.base, self.base]
        self.black_king             = (4, 0)
        self.white_king             = (4, 7)
        self.playing                = True
        self.timer_started          = False
        self.turn                   = 'white'
        self.moves                  = []
{% endhighlight %}

This block returns if the inputted times are empty or invalid. Then it sets the necessary variables for the game to start or restart. 

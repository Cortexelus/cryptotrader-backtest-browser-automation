Backtest While You Sleep
===============================

Browser automation tool for backtesting on Cryptotrader
by EmperorEvil

Hey guys:| I made a cryptotrader backtest automator for Selenium. irefox plugin. Easy to set up. Sharing with the community. Tip your hackers.

It can automate backtests while you sleep. It can help optimize strategies.

It's still a bit faster to backtest by hand, but you can leave it running and give your attention to something else. You could also run several tests at once.  You could +easily+ use this to adjust your bot according to what backtests best in recent history :O (Cryptotrader should make this a feature! How helpful!).

This first example takes maksm's bot for the "No Trend" Market Bot Competition, randomizes its configuration, backtests, observes the final profit, copies the results to a text document, and repeats the process on loop. (I easily left it running overnight, though did not beat his score.)

( http://cryptotraderlab.org/forums/topic/the-no-trend-challenge/
https://cryptotrader.org/topics/119103/no-trend-market-bot-competition-0-5-btc-for-the-winner ) 

SETUP

1. Install Selenium IDE plugin on Firefox. 
http://docs.seleniumhq.org/download/
v2.5.0 latest version as of writing http://release.seleniumhq.org/selenium-ide/2.5.0/selenium-ide-2.5.0.xpi

2. Install Sideflow.js so we can do control structures (loops) in selenium.
- Download from here https://github.com/darrenderidder/sideflow
- https://raw.github.com/darrenderidder/sideflow/master/sideflow.js
- Open Selenium in Firefox. (Mac: Tools > Selenium IDE)
- Open options. (Options > Options...)
- Where it says "Selenium Core extensions (user-extensions.js)", browse for sideflow.js
- OK
- Restart Selenium.

3. Open the "cryptotrader_backtest_maksm_notrend" test case.
- Download it from my github
- https://github.com/Cortexelus/cryptotrader-backtest-browser-automation
- File > Open > "cryptotrader_backtest_maksm_notrend"
- Click either of the play buttons "Play current test case" 
- Observe! 

Let me take you through how the script works, because you will need to make adjustments for your own. 

This code example randomizes configuration. A better optimization would use simulated annealing or genetic algorithms. 

These are the lines of interest, with descriptions of how the code works:

	open /backtests/dPxcod4JrrkmGY468
	// this opens your backtest page, pre-set your market, time period, etc. 

	storeEval 7.4 + 0.02*(Math.random()-0.5) bull_mt
	// this is how you store variables.
	// this is the same thing in javascript as:
	// bull_mt = 7.4 + 0.02*(Math.random()-0.5) 
	// What I'm doing is creating a random variation on the initial number. It will return something between 7.38-7.42. 

I do a bunch of storeEvals to randomize the configuration a bit. Finally I store all those vars into the variable "param", using ${bull_mt} syntax to refer to them, which looks like this:

	"context.config_bull = new Config(${config_bull})\\${KEY_ENTER}context.config_bear = new Config(${config_bear})\\${KEY_ENTER}context.bull_market_threshold = ${bull_mt}\\${KEY_ENTER}context.bear_market_threshold = ${bear_mt}\\${KEY_ENTER}context.market_short = ${m_s}\\${KEY_ENTER}context.market_long = ${m_l}"
	
It's tricky. Let me explain. Because the code lives an ACE code editor, it doesn't play like a regular textarea. You can't just send it whatever contents you want. You must have selenium send each key command one at a time. 

The \\${KEY_ENTER} is the multi-line string syntax in javascript. It is a backslash followed by hitting the enter key. This becomes: 

	"context.config_bull = new Config(${config_bull})\
	context.config_bear = new Config(${config_bear})\
	context.bull_market_threshold = ${bull_mt}\
	context.bear_market_threshold = ${bear_mt}\
	context.market_short = ${m_s}\
	context.market_long = ${m_l}"

And if you want to delete all the existing code, you do:
	
	sendKeys css=textarea ${KEY_META}a${KEY_META}${KEY_BACKSPACE}

On a Mac, this will select all and delete.
On a different machine, change ${KEY_META} to ${KEY_CTRL}.
Or try ${KEY_COMMAND}, it might work on all systems.

The ${KEY_META} is there twice. The first is key down, the second is key up. It's the same as holding down the command key, hitting A, then releasing the command key. This should work for KEY_SHIFT, KEY_ALT, and KEY_CTRL too.

Okay! So we deleted all the code. Now we type in the new bot code. First take your bot code, and replace the parameters the variables you set earlier. For example: 
	
	context.bear_market_threshold = ${bear_mt}
	context.market_short = ${m_s}
	context.market_long = ${m_l}

Good! Next we need to do some formatting. 

maskm's bot begins like this:
	
	# Ichimoku + Heikin-Ashi http://cryptotraderlab.org mod
	# MtGox 2hr
	
	class Ichimoku
	    constructor: (@tenkan_n, @kijun_n, @senkou_a_n, @senkou_b_n, @chikou_n) ->
	    @tenkan = 0.0
	    @kijun = 0.0

Find a text editor and do the following find+replace: turn all line breaks into "\n${KEY_META}${KEY_BACKSPACE}${KEY_META}"

You only have one line in which to fit all the code, so line breaks must become \n. Furthermore, when you make a new line in ACE editor, it auto-indents. You need to delete the auto-indent. Ctrl+backspace does this. (change KEY_META to KEY_CTRL if not Mac). 

Now the beginning should look like this:
	
	# Ichimoku + Heikin-Ashi http://cryptotraderlab.org mod\n${KEY_META}${KEY_BACKSPACE}${KEY_META}# MtGox 2hr\n\n${KEY_META}${KEY_BACKSPACE}${KEY_META}class Ichimoku\n${KEY_META}${KEY_BACKSPACE}${KEY_META}  constructor: (@tenkan_n, @kijun_n, @senkou_a_n, @senkou_b_n, @chikou_n) ->\n${KEY_META}${KEY_BACKSPACE}${KEY_META}    @tenkan = 0.0\n${KEY_META}${KEY_BACKSPACE}${KEY_META}    @kijun = 0.0\n${KEY_META}${KEY_BACKSPACE}${KEY_META}
	
Good! That's it. 

Next the code clicks "backtest" then "run"

	click css=.btn-backtest
	click css=.btn-run
	
After waiting 10 seconds, it reaches into the results, grabs the last message, and copies the text to the variable backtest_result.  
	
	storeText css=.log div:last-child .message backtest_result

When grabbing divs, you'll have to tinker with firebug to get the css right. If you want to grab the entire log, you can easily click the copy button, then storeText the textarea.

	click css=.btn-copy
	storeText css=.raw-log theWholeLog

OKAY. So you have the results. Now what? You could open a new page and paste the results somewhere. 

For example, this script opens a text editor on socrates.io, clicks at the text area, then types the bot parameters, the backtest result, and a bunch of KEY_ENTER for line breaks:

	open http://socrates.io/#cryptotrader-backtest
	wait 1000
	clickAt css=.CodeMirror-measure
	sendKeys css=.CodeMirror textarea ${params}${KEY_ENTER}${backtest_result}${KEY_ENTER}${KEY_ENTER}${KEY_ENTER}

Finally, we loop back to the beginning, and run it again!

	gotoLabel start

At this point, a more clever script would optimize the parameters of next round based on the success of previous rounds. 

! WORD OF ADVICE !  

Avoid abusing the backtester. This will put a strain on the Cryptotrader servers, which slows everything down for everybody. If you want to go super crazy with optimization, try building a client side backtester with TA-lib. 

Guys, when optimizing, avoiding falling into the trap of overfitting! If you tweak many parameters to do extremely well on one time period, it will probably do suboptimal or awful on another time period. Split your backtest into Training set and Test set. Try splitting it 60/40. Optimize on 60% of the time, and then test it on 40% when you're done. Work with simpler bots.

If we want more dynamic cryptotrader bots, until we have a better api, or a client side setup, browser automation is one way to go. There are tons of cool things you could do with this. I plan to experiment adjusting my bot according to other forecasting factors -- news, search engine volume, NLP sentiment analysis, predicting bull/bear from chat rooms, forums, etc.

Thank you. Tip your hackers:|

	BTC
	19R3GbKrC6rpFiFYqZ4Wp5FxxnwQpQQkWE

(ps. will code for bitcoin)






import Draw
import time
import random
import math

BALL_SIZE = 40
MIN_HEIGHT = 240
MAX_HEIGHT = 100
TAP_DX = .25
TAP_DY = .25
ROUND_SECONDS = 60
left = 0 
right = 500
            
def Main():
    #start net at lowest height
    netHeight = MIN_HEIGHT
    
    #score starts at zero
    score = 0
    #keep track of your previous score
    prev = 0
    
    #put net on left or right side
    netSide = random.choice([left,right])  
    
    #initialize position of ball
    x = 230
    y = 470
    dx = 0 
    dy = 0
    
    # get the time when game starts
    startTime = time.time()
    
    #Time left to play
    remainingTime = ROUND_SECONDS - (time.time() - startTime) 
    
    #draw the board
    DrawBoard(netHeight,score,remainingTime,netSide,x,y)   
        
   #while time is remaining
    while remainingTime > 0:

        #if player hits a key
        if Draw.hasNextKeyTyped():
            #get the key
            newKey = Draw.nextKeyTyped()
            
            #if net is on the right side
            if netSide == right:
                
                #make balls velocity the constant
                dx = TAP_DX   
                dy = -TAP_DY
                               
           #if net is on the left
            else:
                #make the balls velocity the negative constant
                dx = -TAP_DX
                dy = -TAP_DY 
                                  
            #move the ball based on its velocity
            for i in range(480):
                y += dy
                dx,dy = ballHitRim(x,y,dx,dy,netSide,netHeight)
                dx,dy = ballHitBackboard(x,y,dx,dy,netSide,netHeight)                 
            for i in range(120):
                x += dx 
                
                dx,dy = ballHitRim(x,y,dx,dy,netSide,netHeight)
                dx,dy = ballHitBackboard(x,y,dx,dy,netSide,netHeight)            
                                                                                
            
          #if the ball is not at the bottom of screen
        if y < 470:
        #make the ball start drifting to the right and down ie. gravity
            dx = TAP_DX 
            if netSide == right:
                for i in range(10):
                    x += dx
                    dx,dy = ballHitBackboard(x,y,dx,dy,netSide,netHeight)
                   
            else:
                for i in range(10):
                    x -= dx  
                    dx,dy = ballHitBackboard(x,y,dx,dy,netSide,netHeight)

            for i in range(20):
                dy = TAP_DY
                y += dy
               #if the ball went through the hoop add a point
                if ballWentThroughHoop(x,y, dy, netSide, netHeight) == True:
                    score += 1 
            
                dx,dy = ballHitRim(x,y,dx,dy,netSide,netHeight)
                dx,dy = ballHitBackboard(x,y,dx,dy,netSide,netHeight) 
            
        #if ball goes over the right side of the screen
        if x >= 500:
            #move it the left side of the screen 
            x = -39
            while x < 0:
                x += .25  
            
            #if ball goes over the left side of the screen
        if x + BALL_SIZE < 0:
            #make it come back in from the right side
            x = 500 + BALL_SIZE
            while x > 500:
                x -= .25             

                          
        currentTime = time.time()     
        remainingTime = ROUND_SECONDS - ((currentTime - startTime) * 2)
        
        #if your score adds one then randomize netside
        if score == prev + 1:
            if netSide == left:
                netSide = right
            else:
                netSide = left
            netHeight = random.randint(MAX_HEIGHT,MIN_HEIGHT)
            prev += 1
            
        DrawBoard(netHeight,score,remainingTime,netSide,x,y)
        
    #end game screen   
    Draw.setColor(Draw.BLACK)
    Draw.setFontSize(32)
    Draw.setFontBold(True)
    Draw.string("Game Over",180,200)
    Draw.string("Your score was: " + str(score), 130, 230)
    Draw.show()



def DrawBoard(netHeight,score,remainingTime,netSide,ballX,ballY):
    Draw.clear()
    Draw.setBackground(Draw.GRAY)
    
            
    #draw the backboard
    Draw.setColor(Draw.BLACK)
    Draw.filledRect(netSide,netHeight,10,100)
    
    #draw the rim
    Draw.setColor(Draw.BLACK)
    if netSide == right:
        Draw.rect(netSide - 70, netHeight + 60, 80, 20)
    else:
        Draw.rect(netSide + 5, netHeight + 60, 80, 20)
    
    #Image of Score
    Draw.setFontSize(18)
    Draw.string("Score: " +str(score),120,10)
    
    #Image of countdown
    Draw.string("Time Remaining: ", 240,10)
    Draw.rect(280,30,ROUND_SECONDS, 10)
    Draw.filledRect(280,30,remainingTime,10)
    
    
    #draw the ball
    Draw.setColor(Draw.ORANGE)
    Draw.filledOval(ballX,ballY,BALL_SIZE,BALL_SIZE)   
    
    Draw.show()
    
#function to calculate rim coordinates          
def rimCoords(netSide, netHeight):
    rimBottom = netHeight + 80
    rimTop = netHeight + 60
    if netSide == right:
        rimLeft = netSide - 70
        rimRight = netSide + 10
    else:
        rimLeft = netSide + 5
        rimRight = netSide + 85 
    
    #return the bottom, top, left, and right side of the rim
    return rimBottom, rimTop, rimLeft, rimRight

#function to calculate ball coordinates          
def ballCoords(x,y,netSide, netHeight):
    centerX = x + BALL_SIZE//2
    centerY = y + BALL_SIZE//2 
    
    bottomY = y + BALL_SIZE
    TopY = y    
    
    rightX = x + BALL_SIZE
    
    leftX = x
    
    #return the bottom, top, left, and right side of the ball
    return centerX, centerY, bottomY, TopY, rightX, leftX


#function to check if player scored
def ballWentThroughHoop(x, y, dy, netSide, netHeight):
    #call for rim coordniates   
    rimBottom, rimTop, rimLeft, rimRight = rimCoords(netSide, netHeight)
    centerX, centerY, bottomY, TopY, rightX, leftX  = \
        ballCoords(x,y,netSide, netHeight)
        
    if dy > 0 and centerY == rimBottom and rimLeft <= rightX <= rimRight \
        and rimLeft <= leftX <= rimRight:
        return True
    return False
	
def ballHitRim(x,y,dx,dy,netSide,netHeight):
    #call rim coordinates and ball coordinates
    rimBottom, rimTop, rimLeft, rimRight = rimCoords(netSide, netHeight)
    centerX, centerY, bottomY, TopY, rightX, leftX = \
        ballCoords(x,y,netSide, netHeight)
    
    #if ball is going towards basket on right   
    if dx > 0:
        #if rightmost of ball hits the rim make it change direction
        if rightX >= rimLeft and rimBottom >= bottomY >= rimTop:
            dx = -dx
   
    #if the ball is going towards basket on the left        
    if dx < 0:
        #if left part of ball hits the rim make it change direction
        if leftX <= rimRight and rimTop <= bottomY <= rimBottom:
            dx = -dx
            
    #ball is going up            
    if dy < 0:
        if TopY == rimBottom and rimRight >= centerX >= rimLeft:
            dy = -dy
    
      
    return dx, dy
      

def ballHitBackboard(x,y,dx,dy,netSide,netHeight):
   #call ball coordinates
    centerX, centerY, bottomY, TopY, rightX, leftX = \
        ballCoords(x,y,netSide, netHeight)
    
    #calculate backboard coordinates
    BBBottom = netHeight + 100
    BBTop = netHeight
    BBLeft = netSide
    BBRight = netSide + 10 
    
    #ball is going down
    if dy > 0:
        #the ball is going toward the right and the right side of the ball
        #hits the backboard
        if netSide == right and rightX >= BBLeft and \
           BBTop <= bottomY <= BBBottom:
            dx = -dx
        
        #the ball is going toward the left and the left side of the ball
        #hits the backboard        
        if netSide == left and leftX <= BBRight and \
           BBTop <= bottomY <= BBBottom:
            dx = -dx
        
        if bottomY == BBTop:
            dy =-dy 
    
    return dx,dy

    
Main()
            
        

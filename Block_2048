#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
Block-2048 (a Tetris-y version of 2048)

Students: No changes are necessary in this file.

author: K. Altmann and J. Hollingsworth
last updated: Nov 05 2020
"""

import random
import copy
import ai

# set this to True to see additional messages
show_debug_messages = False

# score increases when blocks combine
score = 0
moves = 0

# create the game board (7 rows, 5 cols)
rows, cols = (7,5)
board = [[0 for col in range(cols)] for row in range(rows)]
changed_cols = []


# return a randomly generated block (power-of-2) between 2-64
def get_block_random():
    return 2**random.randrange(1, 7)


# display the board to the console
def display_board():
    for i in range(rows - 1, -1, -1):
        row_out = ''
        for j in range(0, cols):
            if board[i][j] == 0:
                row_out += '{:<12}'.format('.')
            else:
                row_out += '{:<12}'.format(str(board[i][j]))
        print(row_out)


# read the next block from a file
def get_new_block_from_file():
    return 2


# return the first empty row for a column (-1 if column is full)
def find_first_empty_row(col):
    row = 0
    while row < rows:
        if board[row][col] == 0:
            return row
        else:
            row += 1     
    return -1   # column is full


# given a block and column, try to place the block
def place_block(block, col):
    row = find_first_empty_row(col)
    if row >= 0:
        board[row][col] = block  # place the block
        return True
    return False   # No room left


# check to see if the dropped block is matched with existing blocks
def check_column(col):
    global changed_cols
    row = rows - 1
    while row >= 0:
        matches = 0
        this_block = board[row][col]
        if this_block > 0:
            matches = 0
            
            # look to the left for a match
            if col > 0:
                if board[row][col - 1] == this_block:
                    matches += 2
            
            # look the the right for a match
            if col + 1 < cols:
                if board[row][col + 1] == this_block: #right
                    matches += 4
            
            # look under for a match
            if row > 0:
                if board[row - 1][col] == this_block: #under
                    matches += 1
    
            if matches > 0:
                global score
                score += combine_blocks(row, col, matches)
                changed_cols.append(col)
                
                # need to update left/right cols if they matched
                if (matches & 2) > 0: # left changed
                    changed_cols.append(col-1)
                    impose_gravity(col - 1)
                    
                if (matches & 4) > 0: # right changed
                    changed_cols.append(col+1)
                    impose_gravity(col + 1)
                    
        row -= 1
            

# combine blocks into a single block
# return the value of the new block created
#
#       matches == 7 <---> left/right/under blocks
#       matches == 6 <---> left/right blocks
#       matches == 5 <---> right/under blocks
#       matches == 4 <---> right block
#       matches == 3 <---> left/under blocks
#       matches == 2 <---> left block
#       matches == 1 <---> under block 
#
def combine_blocks(row, col, matches):
    
    this_block = board[row][col]
    
    # 4 block match (on the left, right, and underneath)
    #       X B X
    #         X
    # collapses to the under block (8x value)
    if matches == 7:
        board[row - 1][col] = 8 * this_block    # under
        board[row][col] = 0                     # this block
        board[row][col - 1] = 0                 # left
        board[row][col + 1] = 0                 # right
        return 8 * this_block

    # 3 block match (on the right and left)
    #       X B X
    # collapses to the center block (4x value)
    if matches == 6:
        board[row][col] = 4 * this_block        # this block
        board[row][col - 1] = 0                 # left
        board[row][col + 1] = 0                 # right
        return 4 * this_block
    
    # 3 block match (on the right and underneath)
    #         B X
    #         X
    # collapses to the under block (4x value)
    if matches == 5:
        board[row - 1][col] = 4 * this_block    # under
        board[row][col] = 0                     # this block
        board[row][col + 1] = 0                 # right
        return 4 * this_block
    
    # 2 block match (on the right)
    #         B X
    # collapses to the block (2x value)
    if matches == 4:
        board[row][col] = 2 * this_block        # this block
        board[row][col + 1] = 0                 # right
        return 2 * this_block
    
    # 3 block match (on the left and underneath)
    #       X B
    #         X
    # collapses to the under block (4x value)
    if matches == 3:
        board[row - 1][col] = 4 * this_block    # under
        board[row][col] = 0                     # this block
        board[row][col - 1] = 0                 # left
        return 4 * this_block
    
    # 2 block match (on the left)
    #       X B
    # collapses to the block (2x value)
    if matches == 2:
        board[row][col] = 2 * this_block        # this block
        board[row][col - 1] = 0                 # left
        return 2 * this_block
    
    # 2 block match (underneath)
    #         B
    #         X
    # collapses to the under block (2x value)
    if matches == 1:
        board[row - 1][col] = 2 * this_block    # under
        board[row][col] = 0                     # this block
        return 2 * this_block
            

# drop blocks in a column to the bottom
def impose_gravity(col):
    global changed_cols
    last_empty_row = -1

    a_row = 0
    while a_row < rows:
        if board[a_row][col] == 0:
            if last_empty_row < 0:
                last_empty_row = a_row
            a_row += 1
        else:
            if last_empty_row >= 0:
                for cnt in range(rows - a_row):
                    board[last_empty_row + cnt][col] = board[a_row + cnt][col]
                for cnt in range(last_empty_row + rows - a_row, rows):
                    board[cnt][col] = 0
                last_empty_row = -1
                changed_cols.append(col)
                debug_message("gravity")
            else:
                a_row += 1 
                

# return True if the board is full, otherwise return False
def is_board_full():
    for row in range(rows):
        for col in range(cols):
            if board[row][col] == 0:
                return False
    return True


# reset the game so we can play again
def reset_game():
    global board, score, moves
    board = [[0 for col in range(cols)] for row in range(rows)]
    score = 0
    moves = 0
            
        
# display a debug message if the debug flag is True
def debug_message(msg):
    if show_debug_messages:
        print(msg)

"""
Main code
"""
def main(suppress_output = False, get_block = get_block_random):
    global moves, changed_cols
    
    # preload the on-deck block (this will become the 1st block)
    on_deck = get_block()

    while True:
        # setup the block and on-deck block
        block = on_deck
        on_deck = get_block()

        # ask the AI where to play
        if not suppress_output:
            print("Current Block:", block, "(up next: " + str(on_deck) +  ")")
        b_copy = copy.deepcopy(board)
        b_copy.reverse()
        col = ai.strategy(block, on_deck, b_copy)
        if not suppress_output:
            print("AI: playing a", block, "in column", col)

        moves += 1
    
        debug_message(str(block) + " in " + str(col))
        if not suppress_output:
            print()
    
        # try to place the block on the board
        if place_block(block, col) == False:
            if not suppress_output:
                print("Out of space in column ", col)
                display_board()
                print("Final score:", score)
                print("Moves:", moves)
            return
   
        # made a change the a column (need to check for collapses)
        changed_cols = [col]
        while len(changed_cols) > 0:
            check_column(changed_cols.pop(0))
            
        for col in range(len(board[0])):
            impose_gravity(col)
            
        # show the board
        if not suppress_output:
            display_board()
            print("Score:", score)
            print()
    
        # if the board is full, game over
        if is_board_full():
            if not suppress_output:
                print("Board is full. Game over.")
                print("Final score: ", score)
                print("Moves:", moves)
            return
        

if __name__ == "__main__":
    main()

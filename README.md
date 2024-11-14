# Wise_project
import math
import random
import tkinter as tk
from PIL import Image, ImageTk
import pygame
#import os

# Initialize pygame for sound
pygame.mixer.init()

def play_sound(sound_file):
    try:
        pygame.mixer.music.load(sound_file)
        pygame.mixer.music.play()
    except pygame.error as e:
        print(f"Error playing sound {sound_file}: {e}")

def draw_hexagon(canvas, x, y, size, text):
    angle = math.pi / 3
    coordinates = []
    for _ in range(6):
        coordinates.append((x + size * math.cos(angle), y - size * math.sin(angle)))
        angle += 2 * math.pi / 6
    canvas.create_polygon(coordinates, outline="black", fill="lightyellow")
    canvas.create_text(x, y, text=text, fill="black", font=("Helvetica", 24))

def draw_hexagons(root, word_M, word_B, word_Z, score_label):
    hexagon_frame = tk.Frame(root)
    hexagon_frame.pack(side="left", expand=True, fill="both")

    # Set background image for hexagon_frame
    try:
        bg_img = Image.open("Screenshot 2024-06-04 083313.png")
        screen_width = root.winfo_screenwidth()
        screen_height = root.winfo_screenheight()
        bg_img = bg_img.resize((screen_width, screen_height))
        bg_photo = ImageTk.PhotoImage(bg_img)
        bg_label = tk.Label(hexagon_frame, image=bg_photo)
        bg_label.image = bg_photo
        bg_label.place(x=0, y=0, relwidth=1, relheight=1)
    except IOError as e:
        print(f"Error loading background image: {e}")

    canvas = tk.Canvas(hexagon_frame, width=600, height=600, bg='lightblue')
    canvas.pack()
    radius = 150
    center_x = center_y = 300
    words = ['Z', 'M', 'B']
    inner_text = random.choice(words)
    words.remove(inner_text)

    if inner_text == "Z":
        outer_text = ['o', 'i', 'p', 'g', 'a', 't']
    elif inner_text == "B":
        outer_text = ['a', 'y', 'u', 'n', 'g', 't']
    elif inner_text == "M":
        outer_text = ['n', 'd', 'p', 'a', 'o', 'c']

    canvas.delete("all")
    for i in range(6):
        angle = i * math.pi / 3
        x = center_x + radius * math.cos(angle)
        y = center_y + radius * math.sin(angle)
        draw_hexagon(canvas, x, y, 50, outer_text[i])

    inner_x = center_x
    inner_y = center_y
    inner_size = 80
    draw_hexagon(canvas, inner_x, inner_y, inner_size, inner_text)

    input_frame = tk.Frame(root, bg='lightblue')
    input_frame.pack(side="bottom", pady=10)

    input_label = tk.Label(input_frame, text="Enter your input: ", bg='lightblue', font=("Helvetica", 14))
    input_label.grid(row=0, column=0)

    input_entry = tk.Entry(input_frame, font=("Helvetica", 14), width=20)
    input_entry.grid(row=0, column=1)

    score = tk.IntVar(value=0)

    entered_words_frame = tk.Frame(root, bg='lightblue')
    entered_words_frame.pack(side="right", padx=10)

    entered_words_label = tk.Label(entered_words_frame, text="Entered Words:", bg='lightblue', font=("Helvetica", 16))
    entered_words_label.pack()

    entered_words_list = tk.Listbox(entered_words_frame, bg='lightblue', font=("Helvetica", 16), height=15, width=20)
    entered_words_list.pack()

    button_frame = tk.Frame(root, bg='lightblue')
    button_frame.pack(side="bottom", pady=10)

    def refresh():
        play_sound("refresh.mp3")
        input_entry.delete(0, 'end')
        nonlocal inner_text
        inner_text = random.choice(words)
        words.remove(inner_text)
        score.set(0)
        score_label.config(text="Score: " + str(score.get()))
        entered_words_list.delete(0, 'end')

        for i in range(6):
            angle = i * math.pi / 3
            x = center_x + radius * math.cos(angle)
            y = center_y + radius * math.sin(angle)
            canvas.delete("outer_text_" + str(i))
            if inner_text == "Z":
                outer_text = ['o', 'i', 'p', 'g', 'a', 't']
            elif inner_text == "B":
                outer_text = ['a', 'y', 'u', 'n', 'g', 't']
            elif inner_text == "M":
                outer_text = ['n', 'd', 'p', 'a', 'o', 'c']
            draw_hexagon(canvas, x, y, 50, outer_text[i])
        draw_hexagon(canvas, inner_x, inner_y, inner_size, inner_text)

    refresh_button = tk.Button(button_frame, text="Refresh", command=refresh, font=("Helvetica", 14), width=10)
    refresh_button.grid(row=0, column=0, padx=5)

    def submit_input(entry):
        play_sound("submit.mp3")
        user_input = entry.get()
        nonlocal inner_text
        success = False
        if inner_text == "Z" and user_input in word_Z:
            word_Z.remove(user_input)
            score.set(score.get() + 1)
            entered_words_list.insert(tk.END, user_input)
            success = True
        elif inner_text == "B" and user_input in word_B:
            word_B.remove(user_input)
            score.set(score.get() + 1)
            entered_words_list.insert(tk.END, user_input)
            success = True
        elif inner_text == "M" and user_input in word_M:
            word_M.remove(user_input)
            score.set(score.get() + 1)
            entered_words_list.insert(tk.END, user_input)
            success = True
        if not success:
            retry_window = tk.Toplevel(root)
            retry_window.title("Oooops....!")
            retry_frame = tk.Frame(retry_window, bg='lavender')
            retry_frame.pack(expand=True, fill="both")
            retry_label = tk.Label(retry_frame, text=" This word does not match. Try again ", font=("Helvetica", 30), bg='lavender')
            retry_label.pack(pady=20)
            close_button = tk.Button(retry_frame, text="Close", font=("Helvetica", 14), command=retry_window.destroy)
            close_button.pack(pady=20)

        score_label.config(text="Score: " + str(score.get()))
        entry.delete(0, 'end')

        if score.get() > 6:
            congrats_window = tk.Toplevel(root)
            congrats_window.title("Congratulations!")
            congrats_frame = tk.Frame(congrats_window, bg='lavender')
            congrats_frame.pack(expand=True, fill="both")
            congrats_label = tk.Label(congrats_frame, text="Congratulations! You won the game!", font=("Helvetica", 20), bg='transparent')
            congrats_label.pack(pady=20)
            close_button = tk.Button(congrats_frame, text="Close", font=("Helvetica", 14), command=congrats_window.destroy)
            close_button.pack(pady=20)

    def quit_game():
        play_sound("quit.mp3")
        root.quit()

    submit_button = tk.Button(button_frame, text="Submit", font=("Helvetica", 14), width=10, command=lambda: submit_input(input_entry))
    submit_button.grid(row=0, column=1, padx=5)

    quit_button = tk.Button(button_frame, text="Quit", font=("Helvetica", 14), width=10, command=quit_game)
    quit_button.grid(row=0, column=2, padx=5)

    score_label.pack(pady=10)

def next_page(root, opening_frame):
    opening_frame.pack_forget()
    word_M = ['man', 'mad', 'mac', 'mop', 'map', 'mon', 'mod']
    word_B = ['ban', 'but', 'bun', 'bug', 'bag', 'bat', 'buy']
    word_Z = ['zoo', 'zip', 'zig', 'zag', 'zit', 'zog', 'zot']
    score_label = tk.Label(root, text="Score: 0", bg='lightblue', font=("Helvetica", 16))
    draw_hexagons(root, word_M, word_B, word_Z, score_label)

root = tk.Tk()
root.title("My GUI Application")
root.geometry("800x800")

opening_frame = tk.Frame(root)
opening_frame.pack(expand=True, fill="both")

# Set background image for opening_frame
try:
    opening_bg_image = Image.open("Screenshot 2024-06-04 083732.png")
    screen_width = root.winfo_screenwidth()
    screen_height = root.winfo_screenheight()
    opening_bg_image = opening_bg_image.resize((screen_width, screen_height))
    opening_bg_photo = ImageTk.PhotoImage(opening_bg_image)
    opening_bg_label = tk.Label(opening_frame, image=opening_bg_photo)
    opening_bg_label.image = opening_bg_photo
    opening_bg_label.place(x=0, y=0, relwidth=1, relheight=1)
except IOError as e:
    print(f"Error loading background image: {e}")

opening_label = tk.Label(opening_frame, text="Welcome to \n Polygonal Wordsmith!", font=("Helvetica", 40), bg='dark blue')
opening_label.pack(pady=50)

next_button = tk.Button(opening_frame, text="Next", font=("Helvetica", 14), command=lambda: next_page(root, opening_frame))
next_button.pack()

def show_instructions(root):
    play_sound("refresh.mp3")
    instructions_window = tk.Toplevel(root)
    instructions_window.title("Instructions")
    instructions_window.geometry("800x800")
    instructions_frame = tk.Frame(instructions_window, bg='lavender')
    instructions_frame.pack(expand=True, fill="both")

    instructions_label = tk.Label(instructions_frame, text="How to Play Hexagon Circle:", font=("Helvetica", 20), bg='lavender')
    instructions_label.pack(pady=20)

    instructions_text = tk.Label(instructions_frame, text="1. Click on 'Next' to start the game.\n2. Enter your guess in the input field.\n3. Click 'Submit' to check your guess.\n4. Enjoy the game!", font=("Helvetica", 14), bg='lightblue')
    instructions_text.pack(pady=10)

    close_button = tk.Button(instructions_frame, text="Close", font=("Helvetica", 14), command=instructions_window.destroy)
    close_button.pack(pady=20)

instructions_button = tk.Button(opening_frame, text="Instructions", font=("Helvetica", 14), command=lambda: show_instructions(root))
instructions_button.pack(pady=20)

root.mainloop()

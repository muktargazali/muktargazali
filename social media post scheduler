import tkinter as tk
import customtkinter as ctk
from tkinter import filedialog, messagebox
from datetime import datetime
from PIL import Image, ImageTk
import requests, json, os

class FacebookScheduler:
    def __init__(self, root):
        self.root = root
        self.root.title("Facebook Scheduler")
        self.root.geometry("420x660")
        self.root.resizable(False, False)

        self.PAGE_ID = 'YOUR_PAGE_ID_HERE'  
        self.ACCESS_TOKEN =  'YOUR_API_KEY_HERE'
        
        self.API_URL = f'https://graph.facebook.com/v18.0/{self.PAGE_ID}/photos'

        self.scheduled_posts = []

        self.build_gui()
        self.load_posts()
        self.check_scheduled_posts()


    def build_gui(self):
        main = ctk.CTkFrame(self.root)
        main.pack(fill="both", expand=True, padx=10, pady=10)

        # Create Post Section
        ctk.CTkLabel(main, text="New Post", font=ctk.CTkFont(size=16, weight="bold")).pack(anchor="w", pady=(0, 5))
        
        self.img_btn = ctk.CTkButton(main, text="Select Image", command=self.select_image)
        self.img_btn.pack(pady=(0, 5))
        self.img_label = ctk.CTkLabel(main, text="No image selected")
        self.img_label.pack()

        self.caption_entry = ctk.CTkTextbox(main, height=80)
        self.caption_entry.pack(fill="x", pady=(10, 10))

        # Schedule Section
        schedule_frame = ctk.CTkFrame(main, fg_color="transparent")
        schedule_frame.pack(fill="x", pady=(5, 5))

        self.date_entry = ctk.CTkEntry(schedule_frame, placeholder_text="YYYY-MM-DD", width=140)
        self.date_entry.pack(side="left", padx=(0, 5))

        self.time_entry = ctk.CTkEntry(schedule_frame, placeholder_text="HH:MM", width=120)
        self.time_entry.pack(side="left", padx=(5, 0))

        # Buttons
        btn_frame = ctk.CTkFrame(main, fg_color="transparent")
        btn_frame.pack(fill="x", pady=(5, 10))

        ctk.CTkButton(btn_frame, text="Post Now", command=self.post_now).pack(side="left", expand=True, padx=5)
        ctk.CTkButton(btn_frame, text="Schedule", command=self.schedule_post).pack(side="right", expand=True, padx=5)

        # Scheduled Posts
        ctk.CTkLabel(main, text="Scheduled Posts", font=ctk.CTkFont(size=14, weight="bold")).pack(anchor="w", pady=(10, 2))

        self.posts_container = ctk.CTkScrollableFrame(main, height=220)
        self.posts_container.pack(fill="both", expand=False)

    def select_image(self):
        filepath = filedialog.askopenfilename(filetypes=[("Images", "*.png *.jpg *.jpeg")])
        if filepath:
            self.image_path = filepath
            img = Image.open(filepath)
            img.thumbnail((100, 100))
            self.img_preview = ImageTk.PhotoImage(img)
            self.img_label.configure(image=self.img_preview, text="")
            self.img_btn.configure(text="Change Image")

    def check_scheduled_posts(self):
         now = datetime.now()
         to_post = []

         for post in self.scheduled_posts:
             post_time = datetime.strptime(post["scheduled_time"], "%Y-%m-%d %H:%M")
             if now >= post_time:
                to_post.append(post)

         for post in to_post:
             result = self.post_to_facebook(post['image'], post['caption'])
             if 'id' in result:
                 self.scheduled_posts.remove(post)
                 self.save_posts()
                 self.display_posts()

         # Check again after 60 seconds
         self.root.after(60000, self.check_scheduled_posts)

    def post_now(self):
        caption = self.caption_entry.get("1.0", "end").strip()
        if not hasattr(self, 'image_path'):
            messagebox.showerror("Error", "Please select an image")
            return

        result = self.post_to_facebook(self.image_path, caption)
        if 'id' in result:
            messagebox.showinfo("Success", "Posted successfully!")
        else:
            messagebox.showerror("Error", f"Failed to post: {result.get('error', 'Unknown error')}")

    def schedule_post(self):
        caption = self.caption_entry.get("1.0", "end").strip()
        date = self.date_entry.get()
        time = self.time_entry.get()

        if not hasattr(self, 'image_path'):
            messagebox.showerror("Error", "Please select an image")
            return

        try:
            datetime.strptime(f"{date} {time}", "%Y-%m-%d %H:%M")
        except ValueError:
            messagebox.showerror("Error", "Invalid date/time format")
            return

        post = {
            "image": self.image_path,
            "caption": caption,
            "scheduled_time": f"{date} {time}",
            "status": "Scheduled"
        }

        self.scheduled_posts.append(post)
        self.save_posts()
        self.display_posts()
        messagebox.showinfo("Success", "Post scheduled successfully!")

    def post_to_facebook(self, image_path, message):
        try:
            with open(image_path, 'rb') as image_file:
                files = {'source': image_file}
                data = {'access_token': self.ACCESS_TOKEN, 'message': message}
                response = requests.post(self.API_URL, files=files, data=data)
                return response.json()
        except Exception as e:
            return {'error': str(e)}

    def display_posts(self):
        for widget in self.posts_container.winfo_children():
            widget.destroy()

        if not self.scheduled_posts:
            ctk.CTkLabel(self.posts_container, text="No scheduled posts").pack()
            return

        for post in self.scheduled_posts:
            frame = ctk.CTkFrame(self.posts_container)
            frame.pack(fill="x", pady=3, padx=5)

            ctk.CTkLabel(frame, text=f"📅 {post['scheduled_time']}", font=ctk.CTkFont(size=12)).pack(anchor="w", padx=5)
            ctk.CTkLabel(frame, text=post['caption'][:80] + "...", wraplength=300).pack(anchor="w", padx=5, pady=2)

            btns = ctk.CTkFrame(frame, fg_color="transparent")
            btns.pack(anchor="e", pady=3)

            ctk.CTkButton(btns, text="Post Now", width=80, command=lambda p=post: self.post_scheduled(p)).pack(side="left", padx=4)
            ctk.CTkButton(btns, text="Delete", width=60, fg_color="red", hover_color="#cc0000", command=lambda p=post: self.delete_post(p)).pack(side="left", padx=4)

    def post_scheduled(self, post):
        result = self.post_to_facebook(post['image'], post['caption'])
        if 'id' in result:
            self.scheduled_posts.remove(post)
            self.save_posts()
            self.display_posts()
            messagebox.showinfo("Success", "Posted successfully!")
        else:
            messagebox.showerror("Error", f"Failed to post: {result.get('error', 'Unknown error')}")

    def delete_post(self, post):
        self.scheduled_posts.remove(post)
        self.save_posts()
        self.display_posts()

    def save_posts(self):
        with open("scheduled_posts.json", "w") as f:
            json.dump(self.scheduled_posts, f)

    def load_posts(self):
        if os.path.exists("scheduled_posts.json"):
            with open("scheduled_posts.json", "r") as f:
                self.scheduled_posts = json.load(f)
        self.display_posts()

if __name__ == "__main__":
    ctk.set_appearance_mode("System")
    ctk.set_default_color_theme("blue")
    root = ctk.CTk()
    app = FacebookScheduler(root)
    root.mainloop()

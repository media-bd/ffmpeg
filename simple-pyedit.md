```python
import os
import subprocess
import urllib.request

# ১. শুধুমাত্র Bold ফন্ট রাখা (Regular ফন্ট থাকলে মুছে ফেলা হবে যাতে ফন্ট ওভারল্যাপ না হয়)
os.makedirs("fonts", exist_ok=True)
reg_font = os.path.join("fonts", "NotoSansBengali-Regular.ttf")
if os.path.exists(reg_font):
    os.remove(reg_font)

bold_font_path = os.path.join("fonts", "NotoSansBengali-Bold.ttf")
bold_url = "https://raw.githubusercontent.com/notofonts/notofonts.github.io/main/fonts/NotoSansBengali/hinted/ttf/NotoSansBengali-Bold.ttf"
if not os.path.exists(bold_font_path):
    print("Downloading NotoSansBengali-Bold.ttf...")
    urllib.request.urlretrieve(bold_url, bold_font_path)

# ২. ইনপুট ভিডিওর দৈর্ঘ্য বের করা
input_video = "কাউখালী রাঙ্গামাটি.mp4"
output_video = "কাউখালী রাঙ্গামাটি উপজেলা প্রশাসন দূর্যোগকালীন কার্যক্রম.mp4"

try:
    res = subprocess.run(
        [
            "ffprobe",
            "-v",
            "error",
            "-show_entries",
            "format=duration",
            "-of",
            "default=noprint_wrappers=1:nokey=1",
            input_video,
        ],
        capture_output=True,
        text=True,
        check=True,
    )
    main_dur = float(res.stdout.strip())
except Exception:
    main_dur = 118.0


def fmt_time(seconds):
    h = int(seconds // 3600)
    m = int((seconds % 3600) // 60)
    s = seconds % 60
    return f"{h}:{m:02d}:{s:05.2f}"


def centered_rounded_rect_ass(w, h, r):
    """(0,0) কেন্দ্র ধরে w প্রস্থ এবং h উচ্চতার রাউন্ডেড বক্স তৈরি করে।"""
    x1, y1 = -int(w / 2), -int(h / 2)
    x2, y2 = int(w / 2), int(h / 2)
    k = int(r * 0.55228475)
    return (
        f"m {x1+r} {y1} "
        f"l {x2-r} {y1} "
        f"b {x2-r+k} {y1} {x2} {y1+r-k} {x2} {y1+r} "
        f"l {x2} {y2-r} "
        f"b {x2} {y2-r+k} {x2-r+k} {y2} {x2-r} {y2} "
        f"l {x1+r} {y2} "
        f"b {x1+r-k} {y2} {x1} {y2-r+k} {x1} {y2-r} "
        f"l {x1} {y1+r} "
        f"b {x1} {y1+r-k} {x1+r-k} {y1} {x1+r} {y1}"
    )


def calc_tight_box_width(text, char_px=28.0, pad_x=110):
    """বাংলা লেখার কার-চিহ্ন বাদ দিয়ে ৬০ ফন্ট সাইজের জন্য সঠিক বক্স প্রস্থ হিসাব করে।"""
    combining_marks = set("ািীুূৃেৈোৌংঃঁ্")
    lines = text.split(r"\N")
    max_effective_len = 0
    for line in lines:
        eff_len = sum(1 for ch in line if ch not in combining_marks)
        if eff_len > max_effective_len:
            max_effective_len = eff_len
    width = int(max_effective_len * char_px + pad_x)
    return max(980, min(width, 1840))


# ৩. ASS সাবটাইটেল তৈরি (ফন্ট ওভারল্যাপ রোধে Bold=0 রাখা হয়েছে, কারণ ফন্ট ফাইলটি নিজেই Bold)
ass_header = """[Script Info]
ScriptType: v4.00+
PlayResX: 1920
PlayResY: 1080
WrapStyle: 2
ScaledBorderAndShadow: yes

[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: IntroMain,Noto Sans Bengali,96,&H00FFFFFF,&H000000FF,&H00000000,&H00000000,0,0,0,0,100,100,0,0,1,0,0,5,40,40,0,1
Style: Shape,Noto Sans Bengali,20,&H00000000,&H000000FF,&H00000000,&H00000000,0,0,0,0,100,100,0,0,1,0,0,7,0,0,0,1
Style: Caption,Noto Sans Bengali,60,&H00FFFFFF,&H000000FF,&H00000000,&H00000000,0,0,0,0,100,100,0,0,1,0,0,5,40,40,0,1
Style: TopHead,Noto Sans Bengali,48,&H0000D7FF,&H000000FF,&H00000000,&H00000000,0,0,0,0,100,100,0,0,1,0,0,5,40,40,0,1

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
"""

# ইন্ট্রো (০ থেকে ৬ সেকেন্ড) - স্থির পজিশনে স্মুথ ফেড-ইন ও ফেড-আউট (কোনো ওভারল্যাপ হবে না)
events = [
    (
        "Dialogue: 0,0:00:00.00,0:00:06.00,IntroMain,,0,0,0,,"
        r"{\fad(600,600)\pos(960,260)\fs100\c&HFFFFFF&}উপজেলা প্রশাসন"
    ),
    (
        "Dialogue: 0,0:00:00.00,0:00:06.00,IntroMain,,0,0,0,,"
        r"{\fad(600,600)\pos(960,380)\fs58\c&H59DEFF&}কাউখালী, রাঙ্গামাটি পার্বত্য জেলা"
    ),
    (
        "Dialogue: 0,0:00:00.00,0:00:06.00,IntroMain,,0,0,0,,"
        r"{\fad(600,600)\pos(960,560)\fs64\c&HFFFFFF&}অতিবৃষ্টি ও পাহাড়ধসে"
        r" ক্ষতিগ্রস্ত কাউখালী উপজেলায়\Nদুর্যোগ মোকাবিলায় উপজেলা প্রশাসনের"
        " গৃহীত কার্যক্রম"
    ),
    (
        "Dialogue: 0,0:00:00.00,0:00:06.00,IntroMain,,0,0,0,,"
        r"{\fad(600,600)\pos(960,760)\fs50\c&HFFCC66&}(৫–১২ জুলাই ২০২৬)"
    ),
]

# ২ লাইনে ভাগ করা ১৩টি ক্যাপশন
captions = [
    (
        r"অতিবৃষ্টি ও পাহাড়ধসে ৫–১২ জুলাই ২০২৬ তারিখে\Nকাউখালী উপজেলার বিভিন্ন"
        r" স্থানে ব্যাপক ক্ষয়ক্ষতি দেখা দিয়েছে।"
    ),
    (
        r"পাহাড়ধসে উপজেলার বিভিন্ন স্থানে আনুমানিক ১২০টি বসতবাড়ি\Nআংশিক ও"
        r" সম্পূর্ণভাবে ক্ষতিগ্রস্ত হয়েছে।"
    ),
    (
        r"ইছামতি খালের পানি বিপৎসীমার ওপর দিয়ে প্রবাহিত হওয়ায়\Nনদীতীরবর্তী"
        r" নিম্নাঞ্চলসমূহ প্লাবিত হয়ে পড়েছে।"
    ),
    (
        r"দুর্যোগকবলিত এলাকা সরেজমিনে পরিদর্শন ও সার্বিক পরিস্থিতি\Nপর্যবেক্ষণ"
        r" করছেন উপজেলা নির্বাহী অফিসার।"
    ),
    (
        r"দুর্গম পাহাড়ি জনপদে ঝুঁকিপূর্ণ বসতি পরিদর্শনকালে\Nস্থানীয়"
        r" বাসিন্দাদের খোঁজখবর নেন ও মতবিনিময় করেন তিনি।"
    ),
    (
        r"দুর্যোগ মোকাবিলায় মাইকিং ও বাড়ি বাড়ি গিয়ে জনসচেতনতা সৃষ্টির\Nপাশাপাশি"
        r" প্রয়োজনীয় সতর্কবার্তা প্রচার করা হচ্ছে।"
    ),
    (
        r"প্রাণহানি এড়াতে ঝুঁকিপূর্ণ বসতবাড়ি থেকে বাসিন্দাদের\Nনিরাপদ"
        r" আশ্রয়কেন্দ্রে সরিয়ে নেওয়া হয়েছে।"
    ),
    (
        r"দুর্যোগ মোকাবিলায় ফায়ার সার্ভিস, পুলিশ, স্বেচ্ছাসেবী সংগঠন\Nও"
        r" জনপ্রতিনিধিদের সমন্বয়ে জরুরি কর্মপরিকল্পনা গৃহীত হয়েছে।"
    ),
    (
        r"উপজেলার ৪টি ইউনিয়নে মোট ১৯টি আশ্রয়কেন্দ্র\Nসার্বক্ষণিক ব্যবহারের"
        r" জন্য প্রস্তুত রাখা হয়েছে।"
    ),
    (
        r"আশ্রয়কেন্দ্রে ১৫০ জন দুর্গত মানুষের নিরাপদ আবাসন এবং\Nপ্রতিদিন তিন"
        r" বেলা পুষ্টিকর ও সুষম খাবার নিশ্চিত করা হয়েছে।"
    ),
    (
        r"আশ্রিত শিশুদের পুষ্টির চাহিদা বিবেচনায় দুধ, কলা, ডিম ও বিস্কুট\Nএবং"
        r" নারীদের মাঝে স্যানিটারি ন্যাপকিন বিতরণ করা হয়েছে।"
    ),
    (
        r"দুর্গতদের স্বাস্থ্যসুরক্ষায় ওরস্যালাইন, পানি বিশুদ্ধকরণ ট্যাবলেট\Nও"
        r" জরুরি ওষুধসহ প্রাথমিক চিকিৎসাসেবা প্রদান অব্যাহত রয়েছে।"
    ),
    (
        r"পাহাড়ধসে বন্ধ হয়ে যাওয়া সড়ক থেকে জরুরি ভিত্তিতে মাটি অপসারণ"
        r" করে\Nযানবাহন চলাচল স্বাভাবিক করা হয়েছে।"
    ),
]

start_offset = 7.0
cap_dur = 9.0
box_height = 166  # ফন্ট সাইজ ৬০-এর জন্য সঠিক উচ্চতা
corner_radius = 30

for i, text in enumerate(captions):
    s = fmt_time(start_offset + i * cap_dur)
    e = fmt_time(start_offset + (i + 1) * cap_dur)

    box_w = calc_tight_box_width(text, char_px=28.0, pad_x=110)
    tight_box_shape = centered_rounded_rect_ass(box_w, box_height, corner_radius)

    # লেয়ার ০: টাইট রাউন্ড ট্রান্সপারেন্ট বক্স (স্মুথ ৫০০ms ফেড-ইন ও ফেড-আউট)
    events.append(
        f"Dialogue: 0,{s},{e},Shape,,0,0,0,,"
        r"{\fad(500,500)\pos(960,940)\c&H000000&\1a&H58&\p1}"
        + tight_box_shape
        + r"{\p0}"
    )
    # লেয়ার ১: বক্সের ঠিক মাঝখানে বড় ফন্টের ক্যাপশন (Fontsize: 60)
    events.append(
        f"Dialogue: 1,{s},{e},Caption,,0,0,0,,"
        r"{\fad(500,500)\pos(960,940)}" + text
    )

# ১০, ১১ ও ১২ নম্বর ক্যাপশনের সময় উপরের হেডিং
top_s = fmt_time(start_offset + 9 * cap_dur)
top_e = fmt_time(start_offset + 12 * cap_dur)
top_text = "আশ্রয়কেন্দ্রে মানবিক সহায়তা ও সেবাপ্রদান"
top_w = calc_tight_box_width(top_text, char_px=24.0, pad_x=90)
top_box_shape = centered_rounded_rect_ass(top_w, 82, 24)

events.append(
    f"Dialogue: 0,{top_s},{top_e},Shape,,0,0,0,,"
    r"{\fad(500,500)\pos(960,72)\c&H000000&\1a&H58&\p1}"
    + top_box_shape
    + r"{\p0}"
)
events.append(
    f"Dialogue: 1,{top_s},{top_e},TopHead,,0,0,0,,"
    r"{\fad(500,500)\pos(960,72)}" + top_text
)

# আউট্রো (মূল ভিডিও শেষ হওয়ার পর ৩.৫ সেকেন্ড)
outro_s = fmt_time(6.0 + main_dur)
outro_e = fmt_time(6.0 + main_dur + 3.5)
events.append(
    f"Dialogue: 0,{outro_s},{outro_e},IntroMain,,0,0,0,,"
    r"{\fad(500,500)\pos(960,480)\fs82\c&HFFFFFF&}উপজেলা প্রশাসন, কাউখালী"
)
events.append(
    f"Dialogue: 0,{outro_s},{outro_e},IntroMain,,0,0,0,,"
    r"{\fad(500,500)\pos(960,600)\fs60\c&H59DEFF&}রাঙ্গামাটি পার্বত্য জেলা"
)

with open("cap.ass", "w", encoding="utf-8") as f:
    f.write(ass_header + "\n".join(events) + "\n")

# ৪. FFmpeg কমান্ড তৈরি ও রান করা (ass ফিল্টারের পরে format=nv12 দেওয়া হয়েছে যাতে QSV তে ঘোস্টিং না হয়)
filter_complex = (
    "[0:v]fps=30,format=yuv420p,setsar=1[v0];"
    "[2:v]scale=1920:1080:force_original_aspect_ratio=decrease,"
    "pad=1920:1080:(ow-iw)/2:(oh-ih)/2,fps=30,format=yuv420p,setsar=1[v1];"
    "[2:a]aresample=44100,aformat=sample_fmts=fltp:channel_layouts=stereo[a1];"
    "[3:v]fps=30,format=yuv420p,setsar=1[v2];"
    "[v0][1:a][v1][a1][v2][4:a]concat=n=3:v=1:a=1[vc][aout];"
    "[vc]ass=cap.ass:fontsdir=fonts,format=nv12[vout]"
)

cmd = [
    "ffmpeg",
    "-y",
    "-f",
    "lavfi",
    "-i",
    "color=c=#0b3d2e:s=1920x1080:d=6:r=30",
    "-f",
    "lavfi",
    "-t",
    "6",
    "-i",
    "anullsrc=channel_layout=stereo:sample_rate=44100",
    "-i",
    input_video,
    "-f",
    "lavfi",
    "-i",
    "color=c=#0b3d2e:s=1920x1080:d=3.5:r=30",
    "-f",
    "lavfi",
    "-t",
    "3.5",
    "-i",
    "anullsrc=channel_layout=stereo:sample_rate=44100",
    "-filter_complex",
    filter_complex,
    "-map",
    "[vout]",
    "-map",
    "[aout]",
    "-c:v",
    "hevc_qsv",
    "-preset",
    "veryslow",
    "-global_quality",
    "26",
    "-r",
    "30",
    "-c:a",
    "aac",
    "-b:a",
    "96k",
    "-movflags",
    "+faststart",
    output_video,
]

print("FFmpeg command ready. Running...")
subprocess.run(cmd, check=True)

# রেন্ডার শেষে cap.ass মুছে ফেলা হচ্ছে যাতে প্লেয়ারে ডাবল সাবটাইটেল না দেখায়
if os.path.exists("cap.ass"):
    os.remove("cap.ass")

print(f"Done! Saved as {output_video}")

```

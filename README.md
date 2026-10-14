# adam-ai
Adam AI chatbot

import streamlit as st
import requests
from datetime import datetime

st.set_page_config(
    page_title="Adam AI | مساعدك الذكي المغربي",
    page_icon="🇲🇦",
    layout="centered",
    initial_sidebar_state="collapsed"
)

st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;500;600;700;800&display=swap');

.stApp {
    background:
        radial-gradient(ellipse at top right, rgba(193,39,45,.17), transparent 42%),
        radial-gradient(ellipse at bottom left, rgba(0,98,51,.17), transparent 38%),
        #07111f;
    color: #eef4ff;
    font-family: 'Cairo', sans-serif;
}
html, body, [class*="css"] {
    font-family: 'Cairo', sans-serif;
}
.block-container {
    max-width: 850px;
    padding-top: 1.4rem;
    padding-bottom: 2rem;
}
.hero {
    padding: 25px 23px;
    border-radius: 24px;
    margin-bottom: 18px;
    background: linear-gradient(125deg, #8f1424, #075b43, #123c64);
    border: 1px solid rgba(255,255,255,.16);
    box-shadow: 0 18px 50px rgba(0,0,0,.20);
}
.hero h1 {
    color: white;
    font-size: clamp(1.8rem, 5vw, 2.5rem);
    margin: 0;
    font-weight: 800;
}
.hero p {
    color: #fff1f2;
    margin: 7px 0 0;
    line-height: 1.9;
}
.pill {
    display: inline-block;
    padding: 5px 12px;
    margin-bottom: 12px;
    border-radius: 999px;
    background: rgba(255,255,255,.14);
    color: white;
    font-size: .85rem;
    border: 1px solid rgba(255,255,255,.18);
}
.flag {
    font-size: 2rem;
    margin-bottom: 8px;
}
div[data-testid="stChatMessage"] {
    border: 1px solid rgba(255,255,255,.10);
    border-radius: 16px;
    background: rgba(255,255,255,.035);
    padding: 12px 14px;
    margin-bottom: 10px;
}
.stTextInput input,
.stTextArea textarea,
.stChatInput textarea {
    border-radius: 12px !important;
}
.stButton > button,
.stDownloadButton > button {
    border-radius: 12px;
    min-height: 44px;
    font-weight: 700;
}
section[data-testid="stSidebar"] {
    background: #0b1727;
    border-left: 2px solid rgba(193,39,45,.35);
}
.small-note {
    color: #9fb0c7;
    font-size: .85rem;
    line-height: 1.8;
}
footer {
    visibility: hidden;
}
</style>
""", unsafe_allow_html=True)

# الواجهة الرئيسية مع علم المغرب 🇲🇦
st.markdown("""
<div class="hero" dir="rtl">
    <div class="flag">🇲🇦</div>
    <span class="pill">✨ المغرب في القلب • مساعدك الذكي</span>
    <h1>🤖 Adam AI 🇲🇦</h1>
    <p>
        مساعدك الذكي المغربي! سول، تعلّم، كتب، برمج،
        واكتشف أفكار جديدة بالدارجة المغربية والعربية ولغات أخرى.
    </p>
</div>
""", unsafe_allow_html=True)

DEFAULT_SYSTEM = (
    "أنت Adam AI، مساعد ذكي وودود. أجب بنفس لغة المستخدم، "
    "واستعمل الدارجة المغربية إذا كتب بها. كن واضحاً ومنظماً، "
    "ولا تخترع معلومات إذا لم تكن متأكداً. "
    "لا تطلب من المستخدم مشاركة كلمات المرور أو المفاتيح السرية."
)

# إعدادات المساعد
with st.sidebar:
    st.markdown("## 🇲🇦 إعدادات Adam AI")

    api_key = st.text_input(
        "مفتاح OpenRouter API",
        type="password",
        help="أدخل المفتاح هنا فقط. لا تضعه داخل الكود أو GitHub.",
        placeholder="sk-or-v1-…"
    )

    model = st.text_input(
        "اسم النموذج",
        value="openrouter/free",
        help="التوفر المجاني يعتمد على النماذج والحصة المتاحة."
    )

    system_prompt = st.text_area(
        "شخصية المساعد",
        value=DEFAULT_SYSTEM,
        height=150
    )

    st.markdown(
        "[🔑 إدارة مفتاح OpenRouter](https://openrouter.ai/keys)"
    )

    st.markdown(
        '<p class="small-note">رسائلك تُرسل إلى مزوّد النموذج عبر الإنترنت. '
        'تجنب إدخال كلمات المرور أو المعلومات الحساسة.</p>',
        unsafe_allow_html=True
    )

# تخزين المحادثة
if "messages" not in st.session_state:
    st.session_state.messages = []

with st.expander("ℹ️ شنو هو Adam AI؟"):
    st.write(
        "Adam AI واجهة دردشة مغربية كتستعمل نموذجاً لغوياً جاهزاً "
        "عبر OpenRouter. ماشي نموذج مدرّب من الصفر."
    )

# عرض الرسائل السابقة
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# خانة السؤال
prompt = st.chat_input("🇲🇦 كتب سؤالك هنا...")

if prompt:
    st.session_state.messages.append({
        "role": "user",
        "content": prompt
    })

    with st.chat_message("user"):
        st.markdown(prompt)

    if not api_key.strip():
        answer = (
            "🔑 دخل مفتاح OpenRouter من الإعدادات الجانبية، "
            "ومن بعد عاود صيفط سؤالك."
        )

        with st.chat_message("assistant"):
            st.warning(answer)

        st.session_state.messages.append({
            "role": "assistant",
            "content": answer
        })

    elif not model.strip():
        answer = "كتب اسم النموذج فالإعدادات قبل ما تعاود المحاولة."

        with st.chat_message("assistant"):
            st.warning(answer)

        st.session_state.messages.append({
            "role": "assistant",
            "content": answer
        })

    else:
        with st.chat_message("assistant"):
            with st.spinner("🇲🇦 Adam AI كيفكر..."):
                try:
                    history = [
                        {
                            "role": m["role"],
                            "content": m["content"]
                        }
                        for m in st.session_state.messages[-20:]
                        if m["role"] in ("user", "assistant")
                    ]

                    response = requests.post(
                        "https://openrouter.ai/api/v1/chat/completions",
                        headers={
                            "Authorization": f"Bearer {api_key.strip()}",
                            "Content-Type": "application/json",
                            "X-Title": "Adam AI Morocco"
                        },
                        json={
                            "model": model.strip(),
                            "messages": [
                                {
                                    "role": "system",
                                    "content": system_prompt.strip() or DEFAULT_SYSTEM
                                },
                                *history
                            ],
                            "temperature": 0.7,
                            "max_tokens": 1200
                        },
                        timeout=90
                    )

                    if not response.ok:
                        try:
                            payload = response.json()
                            error_text = (
                                payload.get("error", {}).get("message")
                                or response.text
                            )
                        except (ValueError, AttributeError):
                            error_text = response.text

                        if response.status_code == 401:
                            error_text = (
                                "المفتاح غير صحيح أو غير صالح. "
                                "راجع مفاتيح OpenRouter."
                            )
                        elif response.status_code == 402:
                            error_text = (
                                "ما كايناش حصة أو رصيد كافي لهذا الطلب. "
                                "تأكد من توفر نموذج مجاني."
                            )
                        elif response.status_code == 429:
                            error_text = (
                                "طلبات كثيرة أو الحصة المجانية سالات مؤقتاً. "
                                "تسنّى شوية وعاود."
                            )
                        elif response.status_code == 404:
                            error_text = (
                                "النموذج غير متاح أو الاسم غير صحيح. "
                                "جرّب openrouter/free أو نموذجاً متاحاً."
                            )

                        raise RuntimeError(
                            f"HTTP {response.status_code}: {error_text}"
                        )

                    data = response.json()
                    choices = data.get("choices", [])

                    if not choices or not choices[0].get(
                        "message", {}
                    ).get("content"):
                        raise RuntimeError(
                            "المزوّد ما رجّع حتى جواب. جرّب مرة أخرى."
                        )

                    answer = choices[0]["message"]["content"]
                    st.markdown(answer)

                except requests.Timeout:
                    answer = (
                        "الطلب خذا وقت طويل. تأكد من الإنترنت "
                        "وعاود المحاولة."
                    )
                    st.error(answer)

                except requests.RequestException:
                    answer = (
                        "وقع مشكل فالاتصال بالإنترنت أو بخدمة OpenRouter. "
                        "عاود المحاولة."
                    )
                    st.error(answer)

                except Exception as exc:
                    answer = (
                        f"ما قدرش Adam AI يجيب الجواب.\n\n"
                        f"التفاصيل: {exc}"
                    )
                    st.error(answer)

        st.session_state.messages.append({
            "role": "assistant",
            "content": answer
        })

# أزرار التحكم
st.divider()

left, right = st.columns(2)

with left:
    if st.button("🧹 مسح المحادثة", use_container_width=True):
        st.session_state.messages = []
        st.rerun()

with right:
    chat_text = "\n\n".join(
        f"{'أنت' if m['role'] == 'user' else 'Adam AI'}: {m['content']}"
        for m in st.session_state.messages
    )

    st.download_button(
        "⬇️ تحميل المحادثة",
        data=chat_text or "ما كايناش محادثة بعد.",
        file_name=f"adam_ai_chat_{datetime.now().strftime('%Y%m%d')}.txt",
        mime="text/plain",
        use_container_width=True
    )

# التوقيع المغربي
st.markdown(
    '<p class="small-note" style="text-align:center;margin-top:16px" dir="rtl">'
    '🇲🇦 Adam AI • صنع بحب في المغرب • المغرب في القلب 🇲🇦</p>',
    unsafe_allow_html=True
)

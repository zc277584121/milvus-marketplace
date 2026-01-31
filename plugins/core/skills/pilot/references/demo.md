# Demo Guide

Quickly build demos to showcase development results.

## Tool Selection

| Tool | Features | Use Case |
|------|----------|----------|
| Gradio | Simplest, few lines of code | Quick demos |
| Streamlit | Feature-rich, polished | Internal tools |
| FastAPI + Frontend | Flexible, customizable | Production products |

## Gradio Demo

### Semantic Search

```python
import gradio as gr

def search(query, top_k):
    """Search function"""
    results = semantic_search.search(query, limit=top_k)
    output = ""
    for i, r in enumerate(results, 1):
        output += f"**{i}. (Similarity: {r['score']:.3f})**\n{r['text']}\n\n"
    return output

demo = gr.Interface(
    fn=search,
    inputs=[
        gr.Textbox(label="Search Query", placeholder="Enter what you want to search..."),
        gr.Slider(1, 20, value=5, step=1, label="Number of Results")
    ],
    outputs=gr.Markdown(label="Search Results"),
    title="Semantic Search Demo",
    description="Vector-based semantic search that understands your intent, not just keyword matching."
)

demo.launch()
```

### RAG Q&A

```python
import gradio as gr

def qa(question, history):
    """Q&A function"""
    answer = rag_system.query(question)
    history.append((question, answer))
    return history, ""

with gr.Blocks(title="Knowledge Base Q&A") as demo:
    gr.Markdown("# Knowledge Base Q&A Demo")

    chatbot = gr.Chatbot(height=400)
    msg = gr.Textbox(label="Your Question", placeholder="Ask me anything...")
    clear = gr.Button("Clear Chat")

    msg.submit(qa, [msg, chatbot], [chatbot, msg])
    clear.click(lambda: [], None, chatbot)

demo.launch()
```

### Image Search

```python
import gradio as gr

def search_by_image(image):
    """Image-to-image search"""
    results = image_search.search_by_image(image)
    return [r["path"] for r in results[:6]]

def search_by_text(text):
    """Text-to-image search"""
    results = image_search.search_by_text(text)
    return [r["path"] for r in results[:6]]

with gr.Blocks(title="Image Search") as demo:
    gr.Markdown("# Image Search Demo")

    with gr.Tab("Image-to-Image"):
        img_input = gr.Image(type="filepath", label="Upload Image")
        img_btn = gr.Button("Search Similar Images")
        img_output = gr.Gallery(label="Search Results", columns=3)
        img_btn.click(search_by_image, img_input, img_output)

    with gr.Tab("Text-to-Image"):
        text_input = gr.Textbox(label="Describe the image you're looking for")
        text_btn = gr.Button("Search")
        text_output = gr.Gallery(label="Search Results", columns=3)
        text_btn.click(search_by_text, text_input, text_output)

demo.launch()
```

## Streamlit Demo

### Semantic Search

```python
import streamlit as st

st.title("Semantic Search")

query = st.text_input("Search Query", placeholder="Enter what you want to search...")
top_k = st.slider("Number of Results", 1, 20, 5)

if st.button("Search") and query:
    with st.spinner("Searching..."):
        results = semantic_search.search(query, limit=top_k)

    for i, r in enumerate(results, 1):
        with st.container():
            st.markdown(f"**{i}. Similarity: {r['score']:.3f}**")
            st.write(r["text"])
            st.divider()
```

### RAG Q&A

```python
import streamlit as st

st.title("Knowledge Base Q&A")

# Initialize chat history
if "messages" not in st.session_state:
    st.session_state.messages = []

# Display chat history
for msg in st.session_state.messages:
    with st.chat_message(msg["role"]):
        st.write(msg["content"])

# User input
if prompt := st.chat_input("Ask me anything..."):
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.write(prompt)

    with st.chat_message("assistant"):
        with st.spinner("Thinking..."):
            answer = rag_system.query(prompt)
        st.write(answer)

    st.session_state.messages.append({"role": "assistant", "content": answer})
```

### Data Dashboard

```python
import streamlit as st
import pandas as pd

st.title("Search Quality Dashboard")

col1, col2, col3 = st.columns(3)
col1.metric("Recall@10", "78.5%", "+2.3%")
col2.metric("P95 Latency", "45ms", "-5ms")
col3.metric("QPS", "156", "+12")

st.subheader("Latency Distribution")
latency_data = pd.DataFrame({
    "Latency (ms)": [23, 25, 28, 31, 35, 42, 48, 55, 62, 78]
})
st.bar_chart(latency_data)

st.subheader("Recent Searches")
recent = pd.DataFrame({
    "Time": ["10:23:45", "10:23:42", "10:23:38"],
    "Query": ["Python tutorial", "Machine learning", "Vector database"],
    "Latency (ms)": [32, 28, 41],
    "Results": [10, 10, 8]
})
st.dataframe(recent)
```

## FastAPI + Frontend

### API Service

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from pydantic import BaseModel

app = FastAPI(title="Search Service")

class SearchRequest(BaseModel):
    query: str
    limit: int = 10

class SearchResult(BaseModel):
    text: str
    score: float

@app.post("/search", response_model=list[SearchResult])
def search(req: SearchRequest):
    results = semantic_search.search(req.query, limit=req.limit)
    return results

# Static file serving
app.mount("/", StaticFiles(directory="static", html=True), name="static")
```

### Simple Frontend

```html
<!-- static/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Search Demo</title>
    <style>
        body { font-family: sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
        input { width: 70%; padding: 10px; font-size: 16px; }
        button { padding: 10px 20px; font-size: 16px; }
        .result { border: 1px solid #ddd; padding: 15px; margin: 10px 0; border-radius: 8px; }
        .score { color: #666; font-size: 14px; }
    </style>
</head>
<body>
    <h1>Semantic Search</h1>
    <input type="text" id="query" placeholder="Enter search query...">
    <button onclick="search()">Search</button>
    <div id="results"></div>

    <script>
        async function search() {
            const query = document.getElementById('query').value;
            const res = await fetch('/search', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({query, limit: 10})
            });
            const results = await res.json();

            document.getElementById('results').innerHTML = results.map((r, i) => `
                <div class="result">
                    <div class="score">#${i+1} Similarity: ${r.score.toFixed(3)}</div>
                    <div>${r.text}</div>
                </div>
            `).join('');
        }
    </script>
</body>
</html>
```

## Running

```bash
# Gradio
pip install gradio
python demo.py

# Streamlit
pip install streamlit
streamlit run demo.py

# FastAPI
pip install fastapi uvicorn
uvicorn app:app --reload
```

## Demo Checklist

- [ ] Core functionality demonstrable
- [ ] Clean and clear interface
- [ ] Acceptable response time
- [ ] Basic error messages
- [ ] Shareable with others

## Next Steps

After demo is complete, the development cycle ends:
- Return to controller → `pilot`
- Iterate based on feedback

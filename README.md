Talk2Data App: Step-by-Step Guide
Offline AI Data Analyst with Streamlit, Pandas and Ollama (Llama 3.2)
Ye app kya karta hai
Ye ek chhota web app hai. Tum ek CSV file upload karte ho, English me sawaal poochhte ho, aur ek local AI model (Llama via Ollama) pandas code likh deta hai. Wo code tumhare data par chalta hai aur answer screen par aata hai. Sab kuch tumhare laptop par chalta hai, internet ya API key ki zaroorat nahi.
Poora flow: CSV upload, pandas table, sawaal likho, prompt banta hai, Llama pandas code likhta hai, code saaf hota hai, eval se chalta hai, answer screen par.
Steps

Step 1:
Zaroori packages install karna
%pip install streamlit pandas langchain-ollama

•	Is code ke liye sirf ye 3 packages chahiye. matplotlib, seaborn aur openpyxl tab chahiye jab charts ya Excel files use karo.
•	Install ke baad Kernel > Restart karo.
•	Ollama alag se install hona chahiye aur model download hona chahiye (ollama pull llama3.2).

Step 2: 
Code ko file me save karna
%%writefile app.py

•	Ye Jupyter ka command hai aur cell ki sabse pehli line honi chahiye.
•	Matlab: neeche ka poora code app.py naam ki file me save karo. Cell chalane par code chalta nahi, sirf save hota hai.

Step 3: 
Warnings band karna
import warnings
warnings.filterwarnings("ignore")

•	Agar koi chhoti warning aaye to use screen par mat dikhao.

Step 4: 
Libraries import karna
import streamlit as st
import pandas as pd
from langchain_ollama import ChatOllama

•	streamlit as st: website jaisa interface banane ke liye (button, text box, table). Ab st likhne se kaam chalega.
•	pandas as pd: data (CSV) padhne aur analysis ke liye.
•	ChatOllama: apne laptop par chal rahe Ollama model se baat karne ke liye.

Step 5:
Page ka title aur layout set karna
st.set_page_config(page_title="Talk2Data", layout="wide")
st.title("AI Data Analyst Tool")

•	page_title: browser tab me ye naam dikhega.
•	layout="wide": poori screen ki chaudai use hogi.
•	st.title(...): page ke upar badi heading.

Step 6:
Model ko load karna (ek hi baar)
@st.cache_resource
def load_llm():
return ChatOllama(model="llama3.2", temperature=0)
 
llm = load_llm()

•	model="llama3.2": kaun sa model use hoga (naam wahi jo ollama list me dikhe).
•	temperature=0: model ko creative nahi, seedha aur same jawab dene ko kehta hai.
•	@st.cache_resource: model ek baar load hoga, har sawaal par dobara nahi. Isse app tez chalta hai.

Step 7: 
File upload ka button banana
uploaded_file = st.sidebar.file_uploader("Upload your CSV file", type=["csv"])

•	Left sidebar me upload ka box aata hai, jisme sirf CSV file chalegi.
•	File aane tak uploaded_file khaali rehta hai (None).

Step 8:
Check karna ki file aayi ya nahi
if uploaded_file is not None:

•	Agar file upload ho gayi hai to neeche ka sab kaam hoga.
•	Iske neeche wali saari lines thodi andar (indent) likhi hain, kyunki wo isi if ke andar hain.

Step 9:
CSV ko table (DataFrame) me badalna
df = pd.read_csv(uploaded_file)

•	df ek table hai jisme poora data hai. Aage ka code isi df par chalta hai.

Step 10: 
Revenue column banana
if 'Quantity' in df.columns and 'UnitPrice' in df.columns:
df['TotalRevenue'] = df['Quantity'] * df['UnitPrice']

•	Agar file me Quantity aur UnitPrice columns hain, to naya column TotalRevenue banta hai (har row ka quantity x price).
•	Ye isliye kiya taaki model ko revenue ka hisaab khud na karna pade.

Step 11:
Data ki jhalak dikhana
st.subheader("Data Preview")
st.dataframe(df.head())

•	Table ki pehli 5 rows screen par dikhti hain, taaki pata chale file sahi khuli.

Step 12:
Sawaal poochne ka box
user_question = st.text_input("Apna data question poochhein:")

•	Ek text box aata hai. Tum jo type karke Enter dabate ho, wo user_question me aa jata hai.

Step 13:
Sawaal aane par kaam shuru karna
if user_question:
with st.spinner("Analyzing data with Llama..."):
try:

•	if user_question: sirf tab chalo jab kuch likha ho.
•	st.spinner: kaam hone tak Analyzing... ghoomta hua dikhta hai.
•	try: agar neeche kuch galat ho to app crash na ho, except sambhal lega.

Step 14:
Model ko instruction (prompt) likhna
prompt = f"""
You are a Python Pandas Data Analyst.
You are given a DataFrame named `df` with columns: {list(df.columns)}
 
User Question: {user_question}
 
Write ONLY ONE line of valid, executable Python Pandas code to answer the question.
STRICT RULES:
1. Do NOT write any English text or explanations.
2. Use valid Pandas methods like .groupby(), .sum(), .sort_values(), .head(), .value_counts().
3. Do NOT invent invalid syntax.
 
Example 1: df.groupby('Country')['TotalRevenue'].sum().sort_values(ascending=False).head(5)
Example 2: df.groupby('Description')['Quantity'].sum().sort_values(ascending=False).head(5)
"""

•	Ye model ko diya jaane wala message hai: tum pandas analyst ho, ye columns hain, ye sawaal hai, sirf ek line ka code do.
•	{list(df.columns)} aur {user_question} me asli columns aur sawaal apne aap bhar jaate hain (f-string).
•	Example 1 aur 2 model ko dikhate hain ki code kaisa dikhna chahiye.

Step 15: 
Model se jawab mangna
response = llm.invoke(prompt)

•	Prompt model ko bheja jaata hai, aur jo wapas aata hai wo response me hai.
•	Text response.content me milta hai.

Step 16: 
Code ko saaf karna
code_to_exec = response.content.strip().replace("```python", "").replace("```", "").strip()

•	Model kabhi code ko ```python ... ``` ke andar likhta hai.
•	Ye line wo extra chinh aur extra spaces hata kar sirf asli code rakhti hai.

Step 17: 
Sirf pehli line lena
code_lines = [line.strip() for line in code_to_exec.split('\n') if line.strip()]
clean_code = code_lines[0] if code_lines else code_to_exec

•	Code ko lines me todta hai, khaali lines hataata hai, aur sirf pehli line rakhta hai (kyunki prompt me ek line maangi thi).

Step 18: 
Generated code screen par dikhana
st.subheader("Generated Pandas Code:")
st.code(clean_code, language="python")

•	Isse tum dekh sakte ho ki model ne kaunsa code likha, aur sahi hai ya nahi, ye khud check kar sakte ho.

Step 19: 
Code ko chalana
result = eval(clean_code)

•	eval text ko asli Python code ki tarah chalata hai.
•	Isse df.groupby(...) jaisa text pandas ka asli kaam kar deta hai, aur jawab result me aata hai.

Step 20:
Answer dikhana
st.subheader("Answer:")
st.write(result)

•	Result screen par dikhta hai (number ya table).

Step 21: 
Galti ho to error dikhana
except Exception as e:
st.error(f"Execution Error: {e}")

•	Agar model ne galat code likha aur wo chal nahi paya, to crash ke bajaye laal dibbe me error dikhta hai.

Step 22: 
File upload nahi hui to
else:
st.info("Kripya sidebar se apni CSV file upload karein.")

•	Jab tak file nahi aati, ye message dikhta hai.

Step 23: 
App chalana
cd "Python_Class/Best project"
streamlit run app.py

•	Ye Jupyter cell me nahi, Terminal me chalao (File > New > Terminal).
•	Browser me http://localhost:8501 khulega.
•	Band karne ke liye terminal me Ctrl + C dabao.

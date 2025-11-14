# mini-ia-estudiantil
Mini IA para estudiantes: resuelve matemáticas y responde preguntas generales.
from flask import Flask, request, jsonify
from sympy import sympify, SympifyError
import requests, os, time

app = Flask(__name__)

@app.route('/math', methods=['POST'])
def math_solve():
    data = request.get_json() or {}
    expr = data.get('expression', '')
    if not expr:
        return jsonify({"error": "Falta 'expression' en JSON"}), 400
    try:
        result = sympify(expr, evaluate=True)
        return jsonify({"input": expr, "result": str(result)})
    except SympifyError:
        return jsonify({"error": "Expresión inválida"}), 400
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route('/ask', methods=['POST'])
def ask_ai():
    data = request.get_json() or {}
    question = data.get('question', '')
    if not question:
        return jsonify({"error": "Falta 'question'"}), 400

    hf_token = os.environ.get('HF_API_TOKEN')

    if hf_token:
        model = os.environ.get('HF_MODEL', 'google/flan-t5-small')
        url = f"https://api-inference.huggingface.co/models/{model}"
        headers = {"Authorization": f"Bearer {hf_token}"}
        payload = {"inputs": question}
        try:
            r = requests.post(url, headers=headers, json=payload, timeout=30)
            return jsonify(r.json())
        except Exception as e:
            return jsonify({"error": str(e)})
    return jsonify({"answer": "No hay HF_API_TOKEN configurado"})

@app.route('/ping')
def ping():
    return jsonify({"status": "ok", "time": time.time()})

if __name__ == '__main__':
    app.run()

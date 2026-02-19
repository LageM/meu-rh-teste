<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Portal de Informes - RH Digital</title>
    <style>
        :root { --primary: #2563eb; --bg: #f8fafc; }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: var(--bg); display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
        .container { background: white; padding: 2rem; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); width: 100%; max-width: 400px; }
        h2 { margin-top: 0; color: #1e293b; font-size: 1.5rem; text-align: center; }
        .field { margin-bottom: 1.5rem; }
        label { display: block; margin-bottom: 0.5rem; color: #64748b; font-weight: 600; font-size: 0.85rem; }
        input { width: 100%; padding: 0.8rem; border: 2px solid #e2e8f0; border-radius: 8px; font-size: 1rem; transition: border 0.3s; box-sizing: border-box; }
        input:focus { outline: none; border-color: var(--primary); }
        button { width: 100%; padding: 1rem; background: var(--primary); color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: transform 0.2s, background 0.3s; }
        button:hover { background: #1d4ed8; transform: translateY(-2px); }
        #msg { text-align: center; margin-top: 1rem; font-size: 0.9rem; min-height: 1.2rem; }
        .loading { color: var(--primary); font-weight: bold; }
    </style>
</head>
<body>

<div class="container">
    <h2>Informe de Rendimentos</h2>
    
    <div class="field">
        <label>CPF DO FUNCIONÁRIO</label>
        <input type="text" id="cpf" placeholder="000.000.000-00" maxlength="14">
    </div>

    <div class="field">
        <label>ANO CALENDÁRIO</label>
        <input type="number" id="ano" value="2025">
    </div>

    <button onclick="buscarInforme()">BAIXAR DOCUMENTO (PDF)</button>
    <div id="msg"></div>
</div>

<script>
    // Máscara de CPF automática
    const inputCpf = document.getElementById('cpf');
    inputCpf.addEventListener('input', e => {
        let v = e.target.value.replace(/\D/g, "");
        if (v.length > 11) v = v.slice(0, 11);
        v = v.replace(/(\d{3})(\d)/, "$1.$2");
        v = v.replace(/(\d{3})(\d)/, "$1.$2");
        v = v.replace(/(\d{3})(\d{1,2})$/, "$1-$2");
        e.target.value = v;
    });

    async function buscarInforme() {
        const cpf = inputCpf.value;
        const ano = document.getElementById('ano').value;
        const msg = document.getElementById('msg');

        if (cpf.length < 14) {
            msg.style.color = "red";
            msg.innerText = "Digite um CPF válido.";
            return;
        }

        msg.className = "loading";
        msg.innerText = "Localizando páginas do funcionário...";

        try {
            // Chamada para o Fluxo do Fiqon
            const response = await fetch('SUA_URL_WEBHOOK_FIQON', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ cpf, ano })
            });

            if (response.ok) {
                const blob = await response.blob();
                const url = window.URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `Informe_${cpf}_${ano}.pdf`;
                document.body.appendChild(a);
                a.click();
                msg.style.color = "green";
                msg.innerText = "Download concluído!";
            } else {
                msg.style.color = "red";
                msg.innerText = "Informe não encontrado para este CPF/Ano.";
            }
        } catch (err) {
            msg.style.color = "red";
            msg.innerText = "Erro na conexão com o servidor.";
        }
    }
</script>

</body>
</html>

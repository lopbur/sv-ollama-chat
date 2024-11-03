<script lang="ts">
    let text = '';
    let hate_class = '';

    async function handleSubmit(this: HTMLFormElement) {
        try {
            const response = await fetch(
                new Request('http://aoji.mooo.com:8080/hate', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        text: text
                    })
                })
            );

            if (!response.ok) return;
            if (!response.body) return;

            hate_class = (await response.json())?.cls;
        } catch (err) {
            text = `Error: ${err}`;
        }
    }
</script>

<div class="container">
    <div class="form-container">
        <h2>혐오발언 필터링 FORM</h2>
        <textarea bind:value={text} placeholder="여기에 텍스트를 입력하세요..." required></textarea>
        <button on:click={handleSubmit}>번역</button>
    </div>
    <div class="result">
        <h3>번역 결과:</h3>
        <p>{hate_class}</p>
    </div>
</div>

<style>
</style>

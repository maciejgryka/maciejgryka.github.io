## Publish
```bash
git push origin master
```

## Update deps:

```bash
bundle update
git add .
git commit -m "updated deps"
git push origin master
```

## Update rustwasm
```bash
scp rincewind:~/dev/rust-playground/rustwasm/wasm-no-bundler/index.html lab/rustwasm1 && \
scp rincewind:~/dev/rust-playground/rustwasm/wasm-no-bundler/pkg/wasm_no_bundler.js lab/rustwasm1/pkg && \
scp rincewind:~/dev/rust-playground/rustwasm/wasm-no-bundler/pkg/wasm_no_bundler_bg.wasm lab/rustwasm1/pkg
```

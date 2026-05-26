# Flutter AI Skills

Flutter geliştirme sürecini hızlandırmak için hazırlanmış Claude Code skill koleksiyonu.

## Skills

- **flutter-ui-reviewer** — Flutter UI kodunu inceler ve iyileştirme önerileri sunar.
- **flutter-performance-reviewer** — Performans darboğazlarını tespit eder ve optimizasyon önerir.
- **clean-architecture-reviewer** — Clean Architecture prensiplerine uygunluğu denetler.
- **firebase-flutter-helper** — Firebase entegrasyonu için yardımcı skill.

## Yapı

```
flutter-ai-skills/
├── README.md
├── CLAUDE.md                   # LLM davranış kuralları (think / simplicity / surgical / goal-driven)
├── skills/
│   ├── flutter-ui-reviewer/
│   ├── flutter-performance-reviewer/
│   ├── clean-architecture-reviewer/
│   └── firebase-flutter-helper/
└── examples/
    └── analysis_options.yaml   # Önerilen lint kuralları
```

## CLAUDE.md

Tüm skill'ler çalışırken [CLAUDE.md](CLAUDE.md) içindeki 4 temel kuralı baz alır:

1. **Think Before Coding** — varsayımları açıkla, belirsizse sor.
2. **Simplicity First** — asgari kod, istenmeyen feature/abstraction yok.
3. **Surgical Changes** — yalnızca istenen yere dokun, komşu kodu "iyileştirme".
4. **Goal-Driven Execution** — başarı kriteri belirle, doğrulanana kadar döngüye gir.

## Lint kuralları

Skill'lerin temel aldığı `analysis_options.yaml` dosyası `examples/` altında bulunur. Kendi Flutter projenize kopyalayabilirsiniz:

```bash
cp flutter-ai-skills/examples/analysis_options.yaml <projeniz>/analysis_options.yaml
```

## Kullanım

Skill'leri Claude Code üzerinde kullanmak için ilgili klasördeki `SKILL.md` dosyalarını referans alın.



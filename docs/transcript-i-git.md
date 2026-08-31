# Transcript d'incident GitOps — I-GIT

> Trace complète de l'incident, du commit cassé au retour au vert.
> À remplir au fil de l'incident. Chaque entrée commence par `[HH:MM]`.

## Identité de l'incident

| Champ | Valeur |
|-------|--------|
| Référence | I-GIT (commit cassé : tag d'image inexistant `david-9.9.9`) |
| Cluster | `david-eks` |
| Dépôt GitOps | `https://github.com/David35390/david-semishop-gitops` |
| SHA avant incident (`.lab-state/sha-avant.txt`) | `<sha>` |
| SHA du commit cassé | `<sha>` |
| SHA du commit de revert | `<sha>` |
| SHA du merge sur main | `<sha>` |
| Injecté à | `[HH:MM]` |
| Résolu (`Healthy`) à | `[HH:MM]` |
| Qui décide au gate (le merge) | David Mouchère |

## 1. Injection et détection

```text
[HH:MM] inject-commit-casse.sh — commit cassé poussé sur main
        SHA d'avant : <sha> / SHA cassé : <sha>
[HH:MM] Argo CD détecte et synchronise
[HH:MM] Pod inventory-... en ErrImagePull / ImagePullBackOff
        -> <message d'erreur exact>
[HH:MM] Sync Status / Health Status : <états observés>
[HH:MM] Les anciens pods servent-ils encore ? <oui/non + preuve>
```

## 2. Le réflexe d'hier — et ce qu'il devient

```text
[HH:MM] Tentative de réparation directe :
        kubectl set image deployment/inventory api=<image-saine> -n semishop
[HH:MM] selfHeal annule la réparation en <XX> secondes
        -> <sorties avant/après>
[HH:MM] Conclusion : la cause est dans Git, la correction sera dans Git.
```

## 3. Collecte en lecture seule avec le badge lecteur

```text
[HH:MM] kubectl --as=system:serviceaccount:semishop:lecteur -n semishop get pods
        -> <sortie>
[HH:MM] describe pod <pod-en-échec>, section Events
        -> <extrait avec le tag demandé>
[HH:MM] get events --sort-by=.lastTimestamp
        -> <extrait>
```

## 4. Le fait décisif : l'historique Git

```text
[HH:MM] git log --oneline -5
        -> <commit suspect et message>
[HH:MM] git show <sha-suspect>
        -> <diff ancien tag vers david-9.9.9>
[HH:MM] Recoupement : le tag des Events est celui écrit par ce commit.
```

## 5. Diagnostic assistant

```text
[HH:MM] Hypothèse : <hypothèse sourcée par Argo CD, la collecte et le SHA>
[HH:MM] Proposition : branche + git revert <sha> + pull request
        Impact / Risque / Réversibilité : <analyse>
        [VALIDATION HUMAINE REQUISE — le gate est le merge]
```

## 6. Réparation par Git : branche, revert et pull request

```text
[HH:MM] git switch -c fix/revert-bump-casse
[HH:MM] git revert <sha-casse> --no-edit -> SHA du revert : <sha>
[HH:MM] git push -u origin fix/revert-bump-casse
[HH:MM] gh pr create --fill -> URL : <url>
[HH:MM] Run validate sur la PR : <vert/rouge>
```

## 7. Revue de la pull request par l'IA

```text
[HH:MM] Avis IA :

<ce que change le diff / risques / non couvert / FUSIONNER ou NE PAS FUSIONNER>

[HH:MM] Lecture humaine : <accord ou désaccord, avec justification>
```

## 8. GATE — le merge

```text
[HH:MM] Décision : FUSIONNER / NE PAS FUSIONNER — David Mouchère
        Motif : <deux phrases appuyées sur la revue et la lecture humaine>
[HH:MM] Merge exécuté par David Mouchère -> SHA du merge : <sha>
```

## 9. Resynchronisation et retour au vert

```text
[HH:MM] Argo CD détecte le merge et synchronise.
[HH:MM] Application semishop : Synced + Healthy.
[HH:MM] inventory : 2/2 Running, image <tag-saine>.
[HH:MM] Compte rendu consigné dans le journal de bord.
```

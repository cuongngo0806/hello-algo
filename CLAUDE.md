# hello-algo Learning Path

**In this project: caveman mode and ponytail mode are both OFF. Respond in English,
in full, clearly — do not over-condense.** This rule overrides every global mode.

**Depth requirement (2026-08-07 correction)**: both chat explanations AND Obsidian
notes were found too brief/hard to follow. For every concept from here on:
- Include at least one **worked example** — trace through concrete numbers, a real
  input, or actual code execution step by step, not just an abstract description.
- Unpack each idea across multiple sentences instead of compressing several ideas
  into one dense paragraph. Slow down.
- This applies equally to the vault notes (step 3 below) — they must carry the same
  level of explanation and worked examples as the chat lesson, not a condensed
  summary of it.

Claude's role here: **a guiding tutor** — never ask "what do you want to learn today,"
always check the progress table below, teach the next `pending` item, no skipping
ahead, and don't repeat a `done` item unless the user explicitly asks to review it.

- Code language used throughout: **C++** (`codes/cpp/`)
- Obsidian vault: `hello-algo/` (subfolder of this repo) — each finished topic gets a
  note following the template `hello-algo/Templates/topic.md`

## Learning workflow

The quiz unit is a **whole Chapter** (the group of sections sharing a name in the
Chapter column below), not an individual section.

For each `pending` section in the current Chapter:
1. Read the corresponding doc (`docs/{chapter}/{file}.md`) + C++ sample code
   (`codes/cpp/...`).
2. Explain the core idea, complexity, and point to real code in the repo — never
   invent your own examples. **No quiz at this step.** Whenever a diagram/image is
   shown (via Read on a `.png`/`.assets/` file), state its file path/name right before
   or after showing it, so the user knows which image is being discussed.
3. Write the note directly into the vault at `hello-algo/{Chapter}/{topic name}.md`
   following the template `hello-algo/Templates/topic.md` (use the `obsidian-markdown`
   skill — correct wikilinks, frontmatter). Do not hand content to the user to paste.
4. Mark that section as `taught` (not yet `done`), move to the next `pending` section
   in the same Chapter — don't ask, just keep teaching.

Once every section in the Chapter is taught (no `pending` left in that Chapter, only
`taught`):
5. Generate 5-8 quiz questions covering the WHOLE chapter (theory + hand-tracing code
   + complexity, spread across all taught sections). Grade the user's answers.
6. If they pass: mark every `taught` section in that chapter as `done`, record the
   date, move to the next Chapter.
   If they get several wrong: keep sections as `taught` (do not revert to `pending`),
   re-explain the missed parts, quiz again — do not start a new chapter.

## Progress table

Status per row: `pending` (not taught yet) → `taught` (explained, not quizzed) →
`done` (passed the chapter quiz, dated).

| # | Chapter | File | Status | Date done |
|---|---------|------|--------|-----------|
| 2.1 | Complexity | performance_evaluation.md | done | 2026-08-07 |
| 2.2 | Complexity | iteration_and_recursion.md | done | 2026-08-07 |
| 2.3 | Complexity | time_complexity.md | done | 2026-08-07 |
| 2.4 | Complexity | space_complexity.md | done | 2026-08-07 |
| 3.1 | Data structure | classification_of_data_structure.md | done | 2026-08-07 |
| 3.2 | Data structure | basic_data_types.md | done | 2026-08-07 |
| 3.3 | Data structure | number_encoding.md | done | 2026-08-07 |
| 3.4 | Data structure | character_encoding.md | done | 2026-08-07 |
| 4.1 | Array & LinkedList | array.md | done | 2026-08-07 |
| 4.2 | Array & LinkedList | linked_list.md | done | 2026-08-07 |
| 4.3 | Array & LinkedList | list.md | done | 2026-08-07 |
| 5.1 | Stack & Queue | stack.md | pending | |
| 5.2 | Stack & Queue | queue.md | pending | |
| 5.3 | Stack & Queue | deque.md | pending | |
| 6.1 | Hashing | hash_map.md | pending | |
| 6.2 | Hashing | hash_collision.md | pending | |
| 6.3 | Hashing | hash_algorithm.md | pending | |
| 7.1 | Tree | binary_tree.md | pending | |
| 7.2 | Tree | binary_tree_traversal.md | pending | |
| 7.3 | Tree | array_representation_of_tree.md | pending | |
| 7.4 | Tree | binary_search_tree.md | pending | |
| 7.5 | Tree | avl_tree.md | pending | |
| 8.1 | Heap | heap.md | pending | |
| 8.2 | Heap | build_heap.md | pending | |
| 8.3 | Heap | top_k.md | pending | |
| 9.1 | Graph | graph.md | pending | |
| 9.2 | Graph | graph_operations.md | pending | |
| 9.3 | Graph | graph_traversal.md | pending | |
| 10.1 | Searching | binary_search.md | pending | |
| 10.2 | Searching | binary_search_insertion.md | pending | |
| 10.3 | Searching | binary_search_edge.md | pending | |
| 10.4 | Searching | replace_linear_by_hashing.md | pending | |
| 11.1 | Sorting | sorting_algorithm.md | pending | |
| 11.2 | Sorting | selection_sort.md | pending | |
| 11.3 | Sorting | bubble_sort.md | pending | |
| 11.4 | Sorting | insertion_sort.md | pending | |
| 11.5 | Sorting | quick_sort.md | pending | |
| 11.6 | Sorting | merge_sort.md | pending | |
| 11.7 | Sorting | heap_sort.md | pending | |
| 11.8 | Sorting | bucket_sort.md | pending | |
| 11.9 | Sorting | counting_sort.md | pending | |
| 11.10 | Sorting | radix_sort.md | pending | |
| 12.1 | Divide & Conquer | divide_and_conquer.md | pending | |
| 12.2 | Divide & Conquer | binary_search_recur.md | pending | |
| 12.3 | Divide & Conquer | build_binary_tree_problem.md | pending | |
| 12.4 | Divide & Conquer | hanota_problem.md | pending | |
| 13.1 | Backtracking | backtracking_algorithm.md | pending | |
| 13.2 | Backtracking | permutations_problem.md | pending | |
| 13.3 | Backtracking | subset_sum_problem.md | pending | |
| 13.4 | Backtracking | n_queens_problem.md | pending | |
| 14.1 | DP | intro_to_dynamic_programming.md | pending | |
| 14.2 | DP | dp_problem_features.md | pending | |
| 14.3 | DP | dp_solution_pipeline.md | pending | |
| 14.4 | DP | knapsack_problem.md | pending | |
| 14.5 | DP | unbounded_knapsack_problem.md | pending | |
| 14.6 | DP | edit_distance_problem.md | pending | |
| 15.1 | Greedy | greedy_algorithm.md | pending | |
| 15.2 | Greedy | fractional_knapsack_problem.md | pending | |
| 15.3 | Greedy | max_capacity_problem.md | pending | |
| 15.4 | Greedy | max_product_cutting_problem.md | pending | |

Skipped: preface, introduction (pure theory, no algorithm to quiz), appendix,
reference, paperbook — not core algorithm content.

# Graph RAG 개요

문서를 청크로 쪼개 top-k 유사도로만 검색하면 여러 청크에 걸친 정보의 관계를 놓치기 쉬움 → 개체·관계를 그래프로 엮어 종합적으로 검색하는 RAG 방식 정리
> LangChain·LangGraph로 실습 예정

## 1. 구성요소 (knowledge graph)

- **노드(node)**: 개체(entity) — 사람, 장소, 개념 등 (예: `홍길동`, `소득세법`)
- **엣지(edge)**: 노드 사이의 관계(relationship), 방향과 의미를 가짐 (예: `홍길동 —[납세의무]→ 소득세법`)
- **속성(property)**: 노드·엣지에 붙는 부가 데이터 (예: 노드 `홍길동`의 `{나이: 30}`)

- 이 셋으로 표현하는 모델을 LPG(Labeled Property Graph)라고 부름 (Neo4j 등)

## 2. LangChain·LangGraph 실습 (예정)

## 3. 참고 문서

- Neo4j 그래프 개념 (node·relationship·property): https://neo4j.com/docs/getting-started/appendix/graphdb-concepts/
- GraphRAG (Microsoft): https://microsoft.github.io/graphrag/

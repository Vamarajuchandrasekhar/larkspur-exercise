EXTRA_TOOLS: List[Dict[str, Any]] = [   # ✏️ Build 2, step 2.1: schemas for the tools you add
    {
        "name": "fare_rules",
        "description": (
            "Read the Larkspur Handbook (fare_rules_excerpt.md), the text behind "
            "Larkspur's rules, and quote it back. Use this when the customer asks why "
            "a rule applies or why they are not getting something, for example 'why "
            "don't I get a hotel?', or when you must justify a check_policy outcome "
            "(meal, hotel, rebooking, refund) in the Handbook's own words. Returns the "
            "text of one Handbook section, which you then explain to the customer. "
            "Do not use it to decide eligibility; check_policy does that."
        ),
        "input_schema": {
            "type": "object",
            "properties": {
                "section": {
                    "type": "string",
                    "description": (
                        "The Handbook section number (e.g. '6') or any words from its "
                        "title (e.g. 'care while you wait'). If it matches nothing, the "
                        "tool replies with the list of available sections."
                    ),
                }
            },
            "required": ["section"],
        },
    },
]


def fare_rules(section: str) -> str:
    """Return one section of Larkspur's Handbook, matched by number or title words."""
    import re
    from pathlib import Path
    path = Path(__file__).parent / "data" / "americas" / "fare_rules_excerpt.md"
    parts = re.split(r"(?m)^### ", path.read_text(encoding="utf-8"))[1:]
    q = re.sub(r"^section\s+", "", (section or "").strip().lower()).rstrip(".")
    for p in parts:
        heading = p.splitlines()[0].strip()
        num, _, title = heading.partition(".")
        if q and (q == num.strip() or q in title.lower()):
            return "### " + p.strip()
    for p in parts:  # fallback: a word that appears in the section text, e.g. "hotel"
        if q and q in p.lower():
            return "### " + p.strip()
    return {"error": "No Handbook section matches %r. Available: %s"
            % (section, "; ".join(p.splitlines()[0].strip() for p in parts))}


LOCAL_TOOLS: Dict[str, Any] = {"fare_rules": fare_rules} 

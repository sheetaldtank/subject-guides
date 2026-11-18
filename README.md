[README.md](https://github.com/user-attachments/files/35031255/README.md)
# IIT Gandhinagar Library: Subject Guides

A small, self-hosted website of LibGuides-style subject guides for every IITGN department, centre and subject. It links the library's subscribed resources and the Koha catalogue, and holds online research-support tutorials.

## Who can do what

| | Public visitor | Subject Librarian | Admin |
|---|---|---|---|
| Browse guides, A-Z resources, tutorials, subject experts | ✓ | ✓ | ✓ |
| Edit intro, Koha keywords and publish status of **assigned** guides | | ✓ | ✓ (all guides) |
| Add, rename, reorder and remove **boxes** on assigned guides | | ✓ | ✓ |
| Add resources (new or from the shared A-Z list) to assigned guides | | ✓ | ✓ |
| Add, edit and reorder tutorials on assigned guides | | ✓ | ✓ |
| Edit a shared resource | | Only ones they added, or used only on their guides | ✓ |
| Create guides, rename guides, delete guides or resources | | | ✓ |
| Create logins, set roles, assign guides, reset passwords, deactivate | | | ✓ |
| Site settings (Koha OPAC address, contact email, off-campus note) | | | ✓ |

## Guide page structure

Each guide is a column of **boxes** that staff add, rename, move up or down, and remove. There are four box types:

- **List of resources**: databases, e-journals and so on, picked from the shared A-Z list or added new, each with an optional note for that guide
- **Text**: free text with simple formatting (`## heading`, `- bullet`, `1. step`, `**bold**`, `[link](https://...)`)
- **Tutorials list**: the guide's research-support tutorials
- **Library catalogue search**: a Koha search box
- **Subscribed e-journals finder**: shows how many subscribed titles the guide's subjects cover (Institute and ONOS), with a title/ISSN search and an A-Z link

The sidebar is the same on every guide. It holds the catalogue search, the subject librarian's profile (designation, email, phone, office, appointment link), more ways to get help, related guides (worked out from shared resources) and the off-campus access note. Each page also has *Search this guide*, *Print page* and *Back to top*.

A **Subject experts** page lists every guide next to its librarian(s).

## E-journal subscriptions (Institute + ONOS)

`data/ejournals.xlsx` is the library's title-level listing (18,493 titles: 5,067 Institute, 13,426 ONOS, in 45 collections). It is imported on `init`, and it powers:

- **E-Journals** (`/journals`): every subscribed title. You can search by title, ISSN or publisher, and filter by subject guide, Institute or ONOS, collection, first letter and active status. Each title is badged **IITGN** or **ONOS**.
- **One resource per collection** (for example *American Chemical Society Journals*, marked ONOS, or *JSTOR*, marked Institute) in the A-Z list, with its title count.
- **Guide mapping**: each guide has *E-journal subjects*, which are Scopus subject areas from the listing such as `Chemistry; Materials Chemistry`. Staff can edit them on the guide's edit page. These subjects decide which titles appear on the guide. A collection is placed in the guide's *E-Journals* box when it has 10 or more titles in those subjects, or when at least half of its titles are in them.

**Updating the list**: go to **Settings → E-journal subscription listing** and upload the new .xlsx in the same column layout. You can also run `python app.py import-journals path/to/file.xlsx`. The title list is replaced and new collections are added to guides. Links staff added by hand are kept.

## Starter content

- 23 guides: the 13 departments and 8 centres from <https://iitgn.ac.in/departmentlist>, plus *Research Support and Scholarly Communication* and *Patents, Standards and IPR*
- 45 journal collections from the subscription listing, marked Institute or ONOS
- 25 other resources: institute-subscribed databases and tools (Scopus, Web of Science, SciFinder-n, Reaxys, MathSciNet, Turnitin, Grammarly, EndNote and BIS standards, all marked IITGN), plus open-access sources (PubChem, arXiv, Shodhganga and others) and free tools (Zotero, Overleaf).
- 7 Chemistry research-support tutorials: library resources and services; types of sources and evaluating them; literature search databases; reference managers; a SciFinder-n workshop; writing tools; and avoiding plagiarism with Turnitin. Add video and slides links once the recordings exist.
- The Koha OPAC address is set to `https://catalog.iitgn.ac.in`. You can change it under *Settings*.

## Running it

Requirements: Python 3.9 or newer, plus Flask and openpyxl (openpyxl reads the e-journal listing). The database is a single SQLite file.

```bash
cd subject-guides
python3 -m venv venv && . venv/bin/activate
pip install -r requirements.txt
python app.py init        # creates instance/guides.db and prints the first admin password
python app.py run         # prints http://localhost:5000 and http://<this computer's IP>:5000
```

`run` listens on all network interfaces, so other computers on the campus network can open it at `http://<server IP>:5000`, for example `http://10.0.114.118:5000`. To limit it to this computer only, use `python app.py run 127.0.0.1`. To use another port, use `python app.py run 0.0.0.0 8080`. If other computers can't connect, allow the port through the server's firewall. On Windows: `netsh advfirewall firewall add rule name="Subject Guides" dir=in action=allow protocol=TCP localport=5000`. On Linux: `sudo ufw allow 5000/tcp`.

Sign in at `/login` with the printed `admin` password and change it under **My account**. Then:

1. **Users → New login**: create a Subject Librarian and tick the guides they look after. If you leave the password blank, one is generated and shown once, so you can share it privately.
2. **Settings**: add the library contact email and check the Koha address.
3. Each librarian signs in, fills in their profile, and builds their guides.

Forgotten password: an admin sets a new one on the user's page. From the server shell you can also run `python app.py reset-password <username>`.

## Production hosting (Linux server)

```bash
pip install gunicorn
SUBJECT_GUIDES_HTTPS=1 gunicorn -w 3 -b 127.0.0.1:8000 app:app
```

Put it behind nginx or Apache with HTTPS, for example at `https://library.iitgn.ac.in/guides`. Example systemd unit:

```ini
[Unit]
Description=IITGN Subject Guides
After=network.target

[Service]
User=www-data
WorkingDirectory=/opt/subject-guides
Environment=SUBJECT_GUIDES_HTTPS=1
ExecStart=/opt/subject-guides/venv/bin/gunicorn -w 3 -b 127.0.0.1:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Environment variables:

- `SUBJECT_GUIDES_DB`: path to the database file (default `instance/guides.db`)
- `SUBJECT_GUIDES_SECRET`: session secret (default: generated into `instance/secret_key`)
- `SUBJECT_GUIDES_HTTPS=1`: secure cookies, for use behind HTTPS
- `ADMIN_PASSWORD`: sets the first admin password during `init`, instead of generating one

**Backups**: copy `instance/guides.db`, for example with a nightly `sqlite3 instance/guides.db ".backup /backups/guides-$(date +%F).db"`.

## Security notes

- Passwords are stored as salted hashes (Werkzeug). Every form is protected against CSRF, and staff sessions expire after 8 hours.
- Repeated failed sign-ins for one username are blocked until the server restarts.
- All user text is escaped. Links must be `http(s)://`.

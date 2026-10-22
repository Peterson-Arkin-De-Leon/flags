# pip install requests reportlab
import io, requests
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter
from reportlab.lib.utils import ImageReader

# ISO-style codes used by FlagCDN; special cases use available substitutes
ENTRIES = [
    ("Afghanistan","af"),("Albania","al"),("Algeria","dz"),("Andorra","ad"),
    ("Angola","ao"),("Antigua and Barbuda","ag"),("Argentina","ar"),("Armenia","am"),
    ("Australia","au"),("Austria","at"),("Azerbaijan","az"),("Bahamas","bs"),
    ("Bahrain","bh"),("Bangladesh","bd"),("Barbados","bb"),("Belarus","by"),
    ("Belgium","be"),("Belize","bz"),("Benin","bj"),("Bhutan","bt"),
    ("Bolivia","bo"),("Bosnia and Herzegovina","ba"),("Botswana","bw"),("Brazil","br"),
    ("Brunei","bn"),("Bulgaria","bg"),("Burkina Faso","bf"),("Burundi","bi"),
    ("Cabo Verde","cv"),("Cambodia","kh"),("Cameroon","cm"),("Canada","ca"),
    ("Central African Republic","cf"),("Chad","td"),("Chile","cl"),("China","cn"),
    ("Colombia","co"),("Comoros","km"),("Congo (Republic)","cg"),("Costa Rica","cr"),
    ("Côte d’Ivoire","ci"),("Croatia","hr"),("Cuba","cu"),("Cyprus","cy"),
    ("Czechia","cz"),("DR Congo","cd"),("Denmark","dk"),("Djibouti","dj"),
    ("Dominica","dm"),("Dominican Republic","do"),("Ecuador","ec"),("Egypt","eg"),
    ("El Salvador","sv"),("Equatorial Guinea","gq"),("Eritrea","er"),("Estonia","ee"),
    ("Eswatini","sz"),("Ethiopia","et"),("Fiji","fj"),("Finland","fi"),
    ("France","fr"),("Gabon","ga"),("Gambia","gm"),("Georgia","ge"),
    ("Germany","de"),("Ghana","gh"),("Greece","gr"),("Grenada","gd"),
    ("Guatemala","gt"),("Guinea","gn"),("Guinea-Bissau","gw"),("Guyana","gy"),
    ("Haiti","ht"),("Honduras","hn"),("Hungary","hu"),("Iceland","is"),
    ("India","in"),("Indonesia","id"),("Iran","ir"),("Iraq","iq"),
    ("Ireland","ie"),("Israel","il"),("Italy","it"),("Jamaica","jm"),
    ("Japan","jp"),("Jordan","jo"),("Kazakhstan","kz"),("Kenya","ke"),
    ("Kiribati","ki"),("Kuwait","kw"),("Kyrgyzstan","kg"),("Laos","la"),
    ("Latvia","lv"),("Lebanon","lb"),("Lesotho","ls"),("Liberia","lr"),
    ("Libya","ly"),("Liechtenstein","li"),("Lithuania","lt"),("Luxembourg","lu"),
    ("Madagascar","mg"),("Malawi","mw"),("Malaysia","my"),("Maldives","mv"),
    ("Mali","ml"),("Malta","mt"),("Marshall Islands","mh"),("Mauritania","mr"),
    ("Mauritius","mu"),("Mexico","mx"),("Micronesia","fm"),("Moldova","md"),
    ("Monaco","mc"),("Mongolia","mn"),("Montenegro","me"),("Morocco","ma"),
    ("Mozambique","mz"),("Myanmar","mm"),("Namibia","na"),("Nauru","nr"),
    ("Nepal","np"),("Netherlands","nl"),("New Zealand","nz"),("Nicaragua","ni"),
    ("Niger","ne"),("Nigeria","ng"),("North Korea","kp"),("North Macedonia","mk"),
    ("Norway","no"),("Oman","om"),("Pakistan","pk"),("Palau","pw"),
    ("Panama","pa"),("Papua New Guinea","pg"),("Paraguay","py"),("Peru","pe"),
    ("Philippines","ph"),("Poland","pl"),("Portugal","pt"),("Qatar","qa"),
    ("Romania","ro"),("Russia","ru"),("Rwanda","rw"),("Saint Kitts and Nevis","kn"),
    ("Saint Lucia","lc"),("Saint Vincent and the Grenadines","vc"),("Samoa","ws"),
    ("San Marino","sm"),("São Tomé and Príncipe","st"),("Saudi Arabia","sa"),
    ("Senegal","sn"),("Serbia","rs"),("Seychelles","sc"),("Sierra Leone","sl"),
    ("Singapore","sg"),("Slovakia","sk"),("Slovenia","si"),("Solomon Islands","sb"),
    ("Somalia","so"),("South Africa","za"),("South Korea","kr"),("South Sudan","ss"),
    ("Spain","es"),("Sri Lanka","lk"),("Sudan","sd"),("Suriname","sr"),
    ("Sweden","se"),("Switzerland","ch"),("Syria","sy"),("Tajikistan","tj"),
    ("Tanzania","tz"),("Thailand","th"),("Timor-Leste","tl"),("Togo","tg"),
    ("Tonga","to"),("Trinidad and Tobago","tt"),("Tunisia","tn"),("Türkiye","tr"),
    ("Turkmenistan","tm"),("Tuvalu","tv"),("Uganda","ug"),("Ukraine","ua"),
    ("United Arab Emirates","ae"),("United Kingdom","gb"),("United States","us"),
    ("Uruguay","uy"),("Uzbekistan","uz"),("Vanuatu","vu"),("Venezuela","ve"),
    ("Vietnam","vn"),("Yemen","ye"),("Zambia","zm"),("Zimbabwe","zw"),

    # UN observers, limited recognition, de facto states
    ("Holy See (Vatican City)","va"),("State of Palestine","ps"),
    ("Taiwan (Republic of China)","tw"),("Kosovo","xk"),
    ("Western Sahara (SADR)","eh"),("Somaliland","so"),
    ("Transnistria","md"),("Abkhazia","ge"),("South Ossetia","ge"),
    ("Northern Cyprus","cy"),

    # Territories, dependencies, and special entities
    ("American Samoa","as"),("Anguilla","ai"),("Aruba","aw"),("Bermuda","bm"),
    ("British Virgin Islands","vg"),("Cayman Islands","ky"),("Cook Islands","ck"),
    ("Curaçao","cw"),("Faroe Islands","fo"),("French Polynesia","pf"),
    ("Gibraltar","gi"),("Greenland","gl"),("Guam","gu"),("Guernsey","gg"),
    ("Hong Kong","hk"),("Isle of Man","im"),("Jersey","je"),("Macau","mo"),
    ("Montserrat","ms"),("New Caledonia","nc"),("Niue","nu"),
    ("Norfolk Island","nf"),("Northern Mariana Islands","mp"),("Puerto Rico","pr"),
    ("Saint Helena","sh"),("Saint Pierre and Miquelon","pm"),("Sint Maarten","sx"),
    ("Svalbard","no"),("Tokelau","tk"),("Turks and Caicos Islands","tc"),
    ("U.S. Virgin Islands","vi"),("Wallis and Futuna","wf"),
]

W, H = letter
c = canvas.Canvas("world_flags_full_color.pdf", pagesize=letter)

for name, code in ENTRIES:
    url = f"https://flagcdn.com/w1280/{code}.png"
    r = requests.get(url, timeout=20)
    r.raise_for_status()
    img = ImageReader(io.BytesIO(r.content))

    c.setFillColorRGB(0.06, 0.09, 0.16)
    c.rect(0, 0, W, H, fill=1, stroke=0)

    # Fit flag while preserving aspect ratio
    iw, ih = img.getSize()
    max_w, max_h = W * 0.78, H * 0.52
    scale = min(max_w / iw, max_h / ih)
    fw, fh = iw * scale, ih * scale
    c.drawImage(img, (W - fw) / 2, (H - fh) / 2 - 30,
                width=fw, height=fh, preserveAspectRatio=True, mask="auto")

    c.setFillColorRGB(0.89, 0.91, 0.94)
    c.setFont("Helvetica-Bold", 30)
    c.drawCentredString(W / 2, H - 95, name)

    c.setFillColorRGB(0.58, 0.64, 0.72)
    c.setFont("Helvetica", 16)
    c.drawCentredString(W / 2, H - 125, "Full-color flag")

    c.showPage()

c.save()
print(f"Created world_flags_full_color.pdf with {len(ENTRIES)} pages.")

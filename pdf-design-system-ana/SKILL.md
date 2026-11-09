---
name: PDF Design System ANA
dust_skill_id: skl_ARsg17x4HzzKS
availability: editors
---

# PDF Design System ANA

**User-facing description:** Gera qualquer documento em PDF com a identidade visual da ANA — A Nova Advocacia: fundo escuro, ciano e roxo, cards arredondados, degradês, capa e numeração de seções. Mesmo padrão do playbook "Objeções e Quebras".

**Agent-facing description:** Use sempre que o usuário pedir para gerar, exportar, converter ou formatar um documento em PDF para a ANA / A Nova Advocacia (playbook, relatório, análise de call, proposta comercial, roteiro, material de treinamento, one-pager, checklist), ou quando pedir um PDF "no design system da ANA", "no padrão da ANA" ou "igual ao documento de objeções". Não use para PDFs de terceiros ou peças processuais que precisem de formatação forense tradicional, nem quando o usuário pedir outro formato (DOCX, PPTX).

> Esta skill está disponível apenas para seus editores (`availability: editors`). Requer a skill "Computer" (Python + reportlab).

## Instructions

## Objetivo
Gerar documentos PDF com a identidade visual da ANA — A Nova Advocacia (dark-first, ciano #00F0FF e roxo #6366F1, cards arredondados, linhas em degradê, capa com spotlights). O padrão de referência é o playbook "Objeções e Quebras".

## Processo
1. Habilite a skill **Computer** se ainda não estiver ativa. Todo o trabalho é feito com Python + reportlab no Computer.
2. Obtenha a biblioteca `ana_pdf.py`: se existir `/files/pod/design-system/ana_pdf.py`, copie para `/tmp/ana_pdf.py`; caso contrário, grave em `/tmp/ana_pdf.py` exatamente o código da seção "Biblioteca" abaixo. NUNCA reescreva cores, fontes ou fundo do zero: use sempre a biblioteca.
3. Organize o conteúdo antes de diagramar: capa (overline, título curto, subtítulo de 1 frase, 2–3 badges), seções numeradas, e escolha o componente certo para cada tipo de conteúdo (tabela abaixo).
4. Escreva um script `/tmp/gerar.py` com `sys.path.insert(0,'/tmp')` e `from ana_pdf import *`, monte o documento com `AnaDoc` e salve em `/files/conversation/<Nome_Descritivo>.pdf` (nome sem espaços, nunca sobrescrever o arquivo de origem do usuário).
5. Verifique: `pdfinfo` (páginas), `qpdf --check`, contagem de palavras por página com `pdftotext -f N -l N` para achar páginas quase vazias (menos de ~60 palavras) e, se houver, remova o `new_page=True` da seção seguinte ou reagrupe. Renderize capa e 1–2 páginas internas com `pdftoppm -r 55` e um recorte de um parágrafo de corpo em `-r 150` para conferir o espaçamento entre palavras e linhas (texto não pode parecer apertado); junte numa imagem, inspecione e depois apague as imagens de teste da pasta da conversa.
6. Entregue com a diretiva de preview do arquivo e um resumo curto do que o documento contém. Mencione pendências marcadas como [CONFIRMAR], se houver.

## API da biblioteca (AnaDoc)
- `d=AnaDoc(header="TÍTULO CURTO · TIPO", footer="ANA — A Nova Advocacia · uso interno")`
- `d.cover(title, subtitle, overline, badges=(...))` — capa opcional (omitir em documentos de 1–2 páginas).
- `d.section(title, overline=None, new_page=False)` — seção numerada automaticamente (01, 02…) com overline ciano e linha degradê. Use `new_page=True` só para blocos grandes.
- `d.h3(t)`, `d.p(t)`, `d.small(t)`, `d.bullets([..])`
- `d.card(items, title, kind)` — kinds: `info` (ciano: scripts, falas, quebras), `analogia` (roxo: analogias, insights, conceitos), `alerta` (âmbar: cuidados, [CONFIRMAR]), `erro` (vermelho: proibições, riscos), `sucesso` (verde: resultados, vitórias), `neutro`. `italic=True` para falas/scripts. `ret=True` devolve o card sem inserir (para usar dentro de KeepTogether).
- `d.table(rows, widths_mm)` — primeira linha é cabeçalho; larguras somam 170 mm.
- `d.steps([(1,"..."),(2,"...")])` — passos com badge circular em degradê.
- `d.numbered_title(n, t)` — título de item numerado (ex.: objeção 1, 2, 3).
- `d.label("Retomada", texto, "#10B981")` — rótulo pequeno colorido + texto.
- `d.divider()`, `d.closing(frase)`, `d.page_break()`, `d.build(caminho, title)`. Para elementos customizados use `d.story.append(...)` com `KeepTogether`.

## Regras de estilo
- Texto é markup do reportlab: escape `&` como `&amp;`; use `<b>`, `<i>`, `<br/>` e destaque palavras-chave com `<font color='#00F0FF'>`. Não use gradiente em texto.
- Símbolos → ⚖ ⚠ ✓ ✗ ★ já são tratados por `fx()` nos métodos da biblioteca; ao montar Paragraph manualmente, passe o texto por `fx()`. Emojis coloridos não renderizam: NÃO use emojis.
- **Espaçamento fluido (obrigatório):** o texto precisa respirar. A biblioteca já define espaço entre palavras de 340/1000 em (`WORD_SPACE`), corpo 9,8 pt com entrelinha 15,8 (≈1,6×), 7 pt entre parágrafos, 11 pt entre blocos e padding interno de 14/12 pt nos cards. NUNCA reduza esses valores, NUNCA use alinhamento justificado, NUNCA crie ParagraphStyle próprio com entrelinha menor que 1,5× o tamanho da fonte nem corpo menor que 8,4 pt (exceto rótulos em caixa-alta). Prefira dividir parágrafos longos (mais de ~5 linhas) e quebrar uma tabela larga em duas a apertar o texto para caber.
- Quebra de página: use `KeepTogether` apenas para manter título + primeiro parágrafo juntos; deixe cards e tabelas longas fluírem para a página seguinte, para não gerar grandes vazios no fim da página.
- Hierarquia: uma ideia por card; títulos curtos; no máximo ~3 cards seguidos sem texto corrido ou tabela entre eles.
- Tom ANA: autoridade acessível, urgência digna, sem promessas de resultado, sem jargão tech gratuito.
- As fontes oficiais (Sora/Inter) não estão instaladas; a biblioteca usa Noto Sans como substituta. NÃO tente baixar fontes.

## Biblioteca (ana_pdf.py)
```python
"""ana_pdf.py — gera PDFs no design system da ANA (A Nova Advocacia). Uso: from ana_pdf import *"""
import random
from reportlab.lib.pagesizes import A4
from reportlab.platypus import (BaseDocTemplate, PageTemplate, Frame, Paragraph, Spacer, Table, TableStyle,
    PageBreak, KeepTogether, Flowable, NextPageTemplate)
from reportlab.lib.styles import ParagraphStyle
from reportlab.lib import colors
from reportlab.lib.units import mm
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from reportlab.lib.fonts import addMapping

# Fontes: Noto Sans substitui Sora/Inter (não instaladas). DejaVu só para os símbolos → ⚖ ⚠ ✓ ✗ ★
N="/usr/share/fonts/truetype/noto/"; D="/usr/share/fonts/truetype/dejavu/"
for n,f in [("In","NotoSans-Regular"),("InB","NotoSans-SemiBold"),("InI","NotoSans-Italic"),("InBI","NotoSans-SemiBoldItalic"),
            ("SoSB","NotoSans-SemiBold"),("SoB","NotoSans-Bold"),("SoXB","NotoSans-ExtraBold"),("SoL","NotoSans-Light")]:
    pdfmetrics.registerFont(TTFont(n,N+f+".ttf"))
pdfmetrics.registerFont(TTFont("DV",D+"DejaVuSans.ttf"))
addMapping("In",0,0,"In");addMapping("In",1,0,"InB");addMapping("In",0,1,"InI");addMapping("In",1,1,"InBI")

# --- Espaçamento fluido: alarga o espaço entre palavras (Noto 260 -> 340 /1000) ---
WORD_SPACE=340
for _n in ["In","InB","InI","InBI","SoSB","SoB","SoXB","SoL"]:
    _f=pdfmetrics.getFont(_n); _f.face.charWidths[32]=WORD_SPACE

H=colors.HexColor
VOID=H("#020408"); DEEP=H("#050B11"); SURF=H("#0A1628"); ELEV=H("#0F1E35"); ALT=H("#0C1A2F")
CYAN=H("#00F0FF"); PURP=H("#6366F1"); PURPL=H("#A5B4FC")
WHITE=H("#F8FAFC"); G100=H("#E2E8F0"); G300=H("#94A3B8"); G500=H("#687280")
AMBER=H("#F59E0B"); RED=H("#EF4444"); GREEN=H("#10B981")
BORDER=H("#0B2A3A"); GRID=H("#16263B")
# variantes de card: (fundo, borda, cor do rótulo)
CARD={"info":(H("#0B1F33"),H("#124055"),CYAN), "analogia":(H("#151E42"),H("#3A3F8F"),PURPL),
      "alerta":(H("#221F1C"),H("#6B4A12"),AMBER), "erro":(H("#241A22"),H("#6B2020"),RED),
      "sucesso":(H("#0B2422"),H("#0F5A45"),GREEN), "neutro":(SURF,BORDER,G300)}
SYMS="→←↑↓⚖⚠✓✗★◆"

body=ParagraphStyle("b",fontName="In",fontSize=9.8,leading=15.8,textColor=G100,spaceAfter=7)
small=ParagraphStyle("s",parent=body,fontSize=8.6,leading=13.6,textColor=G300)
quote=ParagraphStyle("q",parent=body,fontName="InI",textColor=WHITE)
h2=ParagraphStyle("h2",fontName="SoSB",fontSize=17,leading=22,textColor=WHITE,spaceAfter=6)
h3=ParagraphStyle("h3",fontName="SoSB",fontSize=12,leading=15.5,textColor=WHITE,spaceBefore=8,spaceAfter=4)
over=ParagraphStyle("ov",fontName="SoL",fontSize=7.6,leading=10,textColor=CYAN,spaceBefore=4,spaceAfter=3)
lab=ParagraphStyle("lab",fontName="InB",fontSize=7.4,leading=10.5,textColor=CYAN,spaceAfter=5)
cell=ParagraphStyle("c",parent=body,fontSize=8.4,leading=12.8,spaceAfter=0)
cellh=ParagraphStyle("ch",parent=cell,fontName="InB",fontSize=7.2,textColor=CYAN)

def fx(t):
    """Envolve símbolos que a Noto não tem na fonte DejaVu. Aplique em todo texto."""
    return "".join(f"<font name='DV'>{c}</font>" if c in SYMS else c for c in t)
def ovl(t): return "&nbsp;".join(t.upper())

class GradLine(Flowable):
    def __init__(s,w=170*mm,h=1.2,fade=True): Flowable.__init__(s); s.w=w; s.h=h; s.fade=fade
    def wrap(s,*a): return s.w,s.h+4
    def draw(s):
        c=s.canv; c.saveState(); p=c.beginPath(); p.rect(0,2,s.w,s.h); c.clipPath(p,stroke=0,fill=0)
        c.linearGradient(0,0,s.w,0,[DEEP,CYAN,PURP,DEEP] if s.fade else [CYAN,PURP],extend=False); c.restoreState()
class NumBadge(Flowable):
    def __init__(s,n): Flowable.__init__(s); s.n=str(n)
    def wrap(s,*a): return 9*mm,9*mm
    def draw(s):
        c=s.canv; c.saveState(); p=c.beginPath(); p.circle(4.5*mm,4.5*mm-1,4.2*mm); c.clipPath(p,stroke=0,fill=0)
        c.linearGradient(0,9*mm,9*mm,0,[CYAN,PURP],extend=True); c.restoreState()
        c.setFillColor(VOID); c.setFont("SoB",11 if len(s.n)<3 else 8); c.drawCentredString(4.5*mm,3.2*mm-1,s.n)

class AnaDoc:
    """doc=AnaDoc(titulo_cabecalho); doc.cover(...); doc.section(...); doc.p(...); doc.build(caminho)"""
    def __init__(s,header="DOCUMENTO", footer="ANA — A Nova Advocacia"):
        s.story=[]; s.n=0; s.header=header.upper(); s.footer=footer
    # ---------- blocos ----------
    def cover(s,title,subtitle="",overline="",badges=()):
        st=s.story; st.append(NextPageTemplate("content")); st.append(Spacer(1,62*mm))
        if overline: st.append(Paragraph(ovl(overline),ParagraphStyle("co",parent=over,fontSize=8.5)))
        st.append(Paragraph(fx(title),ParagraphStyle("ct",fontName="SoXB",fontSize=44 if len(title)<40 else 32,leading=48 if len(title)<40 else 36,textColor=WHITE,spaceAfter=8)))
        st.append(GradLine(80*mm,2.2,False)); st.append(Spacer(1,10))
        if subtitle: st.append(Paragraph(fx(subtitle),ParagraphStyle("cs",fontName="In",fontSize=13,leading=19,textColor=G300,spaceAfter=24)))
        if badges:
            b=Table([[Paragraph(x.upper(),cellh) for x in badges]],colWidths=[(len(x)*2.1+10)*mm for x in badges])
            b.setStyle(TableStyle([("BACKGROUND",(0,0),(-1,-1),H("#06202B")),("ROUNDEDCORNERS",[9]*4),("ALIGN",(0,0),(-1,-1),"CENTER"),
                ("TOPPADDING",(0,0),(-1,-1),4),("BOTTOMPADDING",(0,0),(-1,-1),5)]+[("BOX",(i,0),(i,0),0.7,H("#0E5566")) for i in range(len(badges))]))
            b.hAlign="LEFT"; st.append(b)
        st.append(PageBreak())
    def section(s,title,overline=None,new_page=False):
        s.n+=1
        if new_page: s.story.append(PageBreak())
        else: s.story.append(Spacer(1,11))
        s.story.append(KeepTogether([Paragraph(f"{s.n:02d}&nbsp;&nbsp;·&nbsp;&nbsp;{ovl(overline or title.split(' —')[0].split(':')[0][:38])}",over),
            Paragraph(fx(title),h2),GradLine(60*mm,1.4,False),Spacer(1,6)]))
    def h3(s,t): s.story.append(Paragraph(fx(t),h3))
    def p(s,t,style=None): s.story.append(Paragraph(fx(t),style or body))
    def small(s,t): s.p(t,small)
    def bullets(s,items,color="#00F0FF"):
        for t in items: s.story.append(Paragraph(f"<font color='{color}' size='11'>•</font>&nbsp;&nbsp;"+fx(t),ParagraphStyle("bu",parent=body,leftIndent=10,firstLineIndent=-10)))
    def card(s,items,title=None,kind="info",italic=False,ret=False):
        bg,line,tc=CARD[kind]; inner=[]
        if title: inner.append(Paragraph(fx(title.upper()),ParagraphStyle("bl",parent=lab,textColor=tc)))
        for it in items: inner.append(Paragraph(fx(it),quote if italic else body) if isinstance(it,str) else it)
        t=Table([[inner]],colWidths=[170*mm])
        t.setStyle(TableStyle([("BACKGROUND",(0,0),(-1,-1),bg),("BOX",(0,0),(-1,-1),0.6,line),("ROUNDEDCORNERS",[10]*4),
            ("LEFTPADDING",(0,0),(-1,-1),14),("RIGHTPADDING",(0,0),(-1,-1),14),("TOPPADDING",(0,0),(-1,-1),12),("BOTTOMPADDING",(0,0),(-1,-1),11)]))
        if ret: return t
        s.story.append(t); s.story.append(Spacer(1,11))
    def table(s,rows,widths_mm,head=True):
        data=[[Paragraph(fx(c.upper() if (head and i==0) else c),cellh if (head and i==0) else cell) for c in r] for i,r in enumerate(rows)]
        t=Table(data,colWidths=[w*mm for w in widths_mm],repeatRows=1 if head else 0)
        st=[("BACKGROUND",(0,0),(-1,-1),SURF),("LINEBELOW",(0,0),(-1,-1),0.4,GRID),("VALIGN",(0,0),(-1,-1),"TOP"),
            ("LEFTPADDING",(0,0),(-1,-1),6),("RIGHTPADDING",(0,0),(-1,-1),6),("TOPPADDING",(0,0),(-1,-1),6),("BOTTOMPADDING",(0,0),(-1,-1),7),
            ("BOX",(0,0),(-1,-1),0.6,BORDER),("ROUNDEDCORNERS",[8]*4)]
        if head: st+=[("BACKGROUND",(0,0),(-1,0),ELEV),("LINEBELOW",(0,0),(-1,0),0.8,H("#145063"))]
        st+=[("BACKGROUND",(0,i),(-1,i),ALT) for i in range(2,len(rows),2)]
        t.setStyle(TableStyle(st)); s.story.append(t); s.story.append(Spacer(1,11))
    def steps(s,items):
        """items=[(n, texto)] — lista numerada com badges em degradê"""
        t=Table([[NumBadge(n),Paragraph(fx(x),body)] for n,x in items],colWidths=[13*mm,157*mm])
        t.setStyle(TableStyle([("VALIGN",(0,0),(-1,-1),"TOP"),("BOTTOMPADDING",(0,0),(-1,-1),8),("LEFTPADDING",(0,0),(0,-1),0)])); s.story.append(t)
    def numbered_title(s,n,t):
        hd=Table([[NumBadge(n),Paragraph(fx(t),ParagraphStyle("ot",parent=h3,spaceBefore=0,fontSize=12.5,leading=16))]],colWidths=[12*mm,158*mm])
        hd.setStyle(TableStyle([("VALIGN",(0,0),(-1,-1),"MIDDLE"),("LEFTPADDING",(0,0),(-1,-1),0)])); s.story.append(Spacer(1,4)); s.story.append(hd); s.story.append(Spacer(1,4))
    def label(s,t,text,color="#00F0FF"):
        s.p(f"<font name='InB' color='{color}' size='7.2'>{t.upper()}</font><br/>{text}")
    def divider(s): s.story += [Spacer(1,6),GradLine(),Spacer(1,4)]
    def closing(s,t): s.story += [Spacer(1,10),Paragraph(fx(t),ParagraphStyle("f",parent=body,alignment=1,fontName="SoL",fontSize=13,leading=18,textColor=CYAN))]
    def page_break(s): s.story.append(PageBreak())
    # ---------- páginas ----------
    def _bg(s,c,doc,cover=False):
        W,Hh=A4; c.saveState(); c.setFillColor(DEEP); c.rect(0,0,W,Hh,fill=1,stroke=0)
        def spot(x,y,r,col,a):
            for i in range(28,0,-1): c.setFillColor(col); c.setFillAlpha(a/28); c.circle(x,y,r*i/28,fill=1,stroke=0)
        spot(W*.12,Hh*.95,(150 if cover else 110)*mm,CYAN,.07 if cover else .035)
        spot(W*.92,Hh*.05,(140 if cover else 100)*mm,PURP,.09 if cover else .045)
        rnd=random.Random(doc.page*7)
        for _ in range(70 if cover else 35):
            c.setFillColor(CYAN if rnd.random()<.3 else WHITE); c.setFillAlpha(rnd.uniform(.08,.22))
            c.circle(rnd.uniform(0,W),rnd.uniform(0,Hh),rnd.uniform(.25,.7),fill=1,stroke=0)
        c.setFillAlpha(1); c.restoreState()
    def _brand(s,c,x,y,size):
        c.saveState(); p=c.beginPath(); p.circle(x+size/2,y+size/2-1,size/2); c.clipPath(p,stroke=0,fill=0)
        c.linearGradient(x,y+size,x+size,y,[CYAN,PURP],extend=True); c.restoreState()
        c.setFillColor(WHITE); c.setFont("SoSB",8.5); c.drawString(x+size+5,y+1,"A NOVA ADVOCACIA")
    def _cover(s,c,doc):
        s._bg(c,doc,True); W,Hh=A4; s._brand(c,20*mm,Hh-24*mm,11)
        c.setFillColor(G500); c.setFont("In",8); c.drawString(20*mm,18*mm,"Plataforma de Inteligência Jurídica para Advogados")
        c.drawRightString(W-20*mm,18*mm,"Inteligência · Estratégia · Conexão · Transformação")
    def _content(s,c,doc):
        s._bg(c,doc); W,Hh=A4; s._brand(c,20*mm,Hh-14*mm,7)
        c.setFillColor(G300); c.setFont("SoL",7.5); c.drawRightString(W-20*mm,Hh-13.2*mm,s.header)
        c.saveState(); p=c.beginPath(); p.rect(20*mm,Hh-17.5*mm,W-40*mm,.6); c.clipPath(p,stroke=0,fill=0)
        c.linearGradient(20*mm,0,W-20*mm,0,[CYAN,PURP,DEEP],extend=False); c.restoreState()
        c.setFillColor(G500); c.setFont("In",7.5); c.drawString(20*mm,11*mm,s.footer)
        c.setFillColor(CYAN); c.setFont("SoSB",8); c.drawRightString(W-20*mm,11*mm,f"{doc.page:02d}")
    def build(s,path,title="ANA"):
        d=BaseDocTemplate(path,pagesize=A4,title=title,author="A Nova Advocacia")
        f=Frame(20*mm,18*mm,170*mm,A4[1]-40*mm,leftPadding=0,rightPadding=0,topPadding=0,bottomPadding=0)
        fc=Frame(20*mm,30*mm,170*mm,A4[1]-60*mm,leftPadding=0,rightPadding=0)
        tpl=[PageTemplate("content",[f],onPage=s._content)]
        if any(isinstance(x,NextPageTemplate) for x in s.story): tpl.insert(0,PageTemplate("cover",[fc],onPage=s._cover))
        d.addPageTemplates(tpl); d.build(s.story); return path
```

## Restrições
- NUNCA mude a paleta, o fundo escuro ou a estrutura de capa/cabeçalho/rodapé, a menos que o usuário peça explicitamente.
- NÃO use fundo branco nem tema claro.
- NÃO invente conteúdo para preencher o layout: o design serve ao conteúdo pedido.

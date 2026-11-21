import sys
import os
import csv
import json
import math
import time
import copy
import platform
import sqlite3
import shutil
import subprocess
import tempfile
import cv2
import numpy as np
from pathlib import Path
PASTA_KRONOS_BASE = Path.home() / "Kronos_Pro"
PASTA_PROJETOS = PASTA_KRONOS_BASE / "Projetos"
PASTA_EXPORTACOES = PASTA_KRONOS_BASE / "Exportacoes"
PASTA_AUTOSAVE = PASTA_KRONOS_BASE / "AutoSave"

for _p in (PASTA_PROJETOS, PASTA_EXPORTACOES, PASTA_AUTOSAVE):
    _p.mkdir(parents=True, exist_ok=True)

AUTOSAVE_PATH = PASTA_AUTOSAVE / "sessao_ativa_autosave.json"
RECENTES_PATH = PASTA_KRONOS_BASE / "projetos_recentes.json"

os.environ["QT_ENABLE_HIGHDPI_SCALING"] = "1"
os.environ["QT_AUTO_SCREEN_SCALE_FACTOR"] = "1"

from PyQt6.QtWidgets import (
    QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout,
    QGridLayout, QPushButton, QLabel, QLineEdit, QSlider, QSplitter,
    QTabWidget, QScrollArea, QFrame, QFileDialog, QMessageBox,
    QCheckBox, QComboBox, QTextEdit, QDialog, QProgressDialog,
    QFormLayout, QSpinBox, QDoubleSpinBox, QColorDialog, QMenu,
    QSizePolicy, QInputDialog, QStackedWidget, QListWidget, QListWidgetItem
)
from PyQt6.QtMultimedia import QMediaPlayer, QAudioOutput
from PyQt6.QtMultimediaWidgets import QVideoWidget
from PyQt6.QtCore import Qt, QUrl, QPointF, QPoint, QRect, QRectF, QSize, QTimer, QThread, pyqtSignal, QEvent
from PyQt6.QtGui import (
    QPainter, QColor, QPen, QFont, QBrush, QImage, QPixmap,
    QIcon, QPolygonF, QAction, QLinearGradient, QPainterPath,
    QPdfWriter, QPageSize, QPageLayout
)

cv2.setUseOptimized(True)
cv2.setNumThreads(4)

APP_NAME = "KRONOS PRO"
APP_VERSION = "0.9.0-beta.1"
APP_SUBTITLE = "Tactical Video & Match Intelligence"


def gerar_icone_app_k(tamanho=256):
    """Cria o ícone da aplicação com um K estilizado."""
    pixmap = QPixmap(tamanho, tamanho)
    pixmap.fill(QColor("#050508"))

    painter = QPainter(pixmap)
    painter.setRenderHint(QPainter.RenderHint.Antialiasing)

    gradient = QLinearGradient(0, 0, tamanho, tamanho)
    gradient.setColorAt(0.0, QColor("#38BDF8"))
    gradient.setColorAt(1.0, QColor("#0284C7"))
    painter.setPen(QPen(QBrush(gradient), max(12, tamanho // 12)))

    margem = tamanho // 4
    painter.drawLine(margem, margem, margem, tamanho - margem)
    painter.drawLine(margem, tamanho // 2, tamanho - margem, margem)
    painter.drawLine(margem, tamanho // 2, tamanho - margem, tamanho - margem)

    painter.end()
    return QIcon(pixmap)


STYLESHEET = """
QMainWindow, QWidget {
    background-color: #000000;
    color: #F8FAFC;
    font-family: -apple-system, 'SF Pro Display', 'Helvetica Neue', sans-serif;
    font-size: 12px;
}
QFrame#Panel {
    background-color: #050508;
    border: 1px solid #181822;
    border-radius: 8px;
}
QFrame#SubHeader {
    background-color: #0A0A10;
    border-bottom: 1px solid #1C1C28;
    border-top-left-radius: 7px;
    border-top-right-radius: 7px;
}
QFrame#Header {
    background-color: #000000;
    border-bottom: 1px solid #1E293B;
}
QFrame#TransportBar {
    background-color: #06060A;
    border-top: 1px solid #181824;
    border-bottom-left-radius: 7px;
    border-bottom-right-radius: 7px;
}
QTabWidget::pane {
    border: 1px solid #181822;
    background-color: #000000;
    border-radius: 6px;
}
QTabBar::tab {
    background-color: #08080D;
    color: #71717A;
    padding: 8px 22px;
    margin-right: 4px;
    border: 1px solid #161620;
    border-bottom: none;
    border-top-left-radius: 6px;
    border-top-right-radius: 6px;
    font-weight: bold;
    letter-spacing: 0.6px;
}
QTabBar::tab:selected {
    background-color: #000000;
    color: #38BDF8;
    border-top: 2px solid #38BDF8;
}
QPushButton {
    background-color: #101018;
    color: #E4E4E7;
    border: 1px solid #242434;
    border-radius: 5px;
    padding: 5px 10px;
    font-weight: 600;
}
QPushButton:hover {
    background-color: #191926;
    border-color: #38BDF8;
    color: #FFFFFF;
}
QPushButton#CollapseBtn {
    background-color: #12121C;
    color: #94A3B8;
    border: 1px solid #222232;
    border-radius: 4px;
    padding: 3px 8px;
    font-size: 11px;
}
QPushButton#CollapseBtn:hover {
    background-color: #0284C7;
    color: #FFFFFF;
}
QPushButton#AccentBtn {
    background-color: #0284C7;
    border: 1px solid #38BDF8;
    color: #FFFFFF;
}
QPushButton#AccentBtn:hover {
    background-color: #0369A1;
}
QPushButton#SuccessBtn {
    background-color: #059669;
    border: 1px solid #34D399;
    color: #FFFFFF;
}
QPushButton#WarnBtn {
    background-color: #D97706;
    border: 1px solid #FBBF24;
    color: #FFFFFF;
}
QPushButton#DangerBtn {
    background-color: #DC2626;
    border: 1px solid #F87171;
    color: #FFFFFF;
}
QLineEdit, QTextEdit, QSpinBox, QDoubleSpinBox, QListWidget {
    background-color: #0B0B12;
    border: 1px solid #262638;
    border-radius: 5px;
    padding: 4px 6px;
    color: #FFFFFF;
    font-weight: bold;
}
QListWidget::item {
    padding: 8px;
    border-bottom: 1px solid #181824;
    border-radius: 4px;
}
QListWidget::item:selected {
    background-color: #0284C7;
    color: #FFFFFF;
}
QListWidget::item:hover {
    background-color: #161624;
}
QComboBox {
    background-color: #0D0D15;
    border: 1px solid #28283A;
    border-radius: 5px;
    padding: 4px 8px;
    color: #FFFFFF;
    font-weight: bold;
}
QComboBox:hover {
    border-color: #38BDF8;
}
QComboBox::drop-down {
    border: none;
    width: 18px;
}
QComboBox QAbstractItemView {
    background-color: #0D0D15;
    color: #FFFFFF;
    selection-background-color: #0284C7;
    selection-color: #FFFFFF;
    border: 1px solid #38BDF8;
    padding: 4px;
}
QMenu {
    background-color: #0B0B12;
    color: #F8FAFC;
    border: 1px solid #38BDF8;
    border-radius: 6px;
    padding: 4px;
}
QMenu::item {
    padding: 6px 20px;
    border-radius: 4px;
}
QMenu::item:selected {
    background-color: #0284C7;
    color: #FFFFFF;
}
QSlider::groove:horizontal {
    height: 5px;
    background: #181824;
    border-radius: 2px;
}
QSlider::sub-page:horizontal {
    background: #F59E0B;
    border-radius: 2px;
}
QSlider::handle:horizontal {
    background: #FFFFFF;
    border: 2px solid #F59E0B;
    width: 13px;
    margin-top: -4px;
    margin-bottom: -4px;
    border-radius: 6px;
}
QSplitter::handle {
    background-color: #162033;
    border: 1px solid #263449;
}
QSplitter::handle:horizontal {
    width: 10px;
}
QSplitter::handle:vertical {
    height: 10px;
}
QSplitter::handle:hover {
    background-color: #38BDF8;
    border-color: #7DD3FC;
}
QScrollBar:vertical {
    background-color: #070B12;
    width: 11px;
    margin: 2px;
    border-radius: 5px;
}
QScrollBar::handle:vertical {
    background-color: #0EA5E9;
    min-height: 28px;
    border-radius: 5px;
}
QScrollBar::handle:vertical:hover {
    background-color: #7DD3FC;
}
QScrollBar::horizontal {
    background-color: #070B12;
    height: 11px;
    margin: 2px;
    border-radius: 5px;
}
QScrollBar::handle:horizontal {
    background-color: #0EA5E9;
    min-width: 28px;
    border-radius: 5px;
}
QScrollBar::handle:horizontal:hover {
    background-color: #7DD3FC;
}
QScrollBar::add-line, QScrollBar::sub-line {
    width: 0px;
    height: 0px;
}
"""


# =============================================================================
# GESTOR DE LICENÇA (OFFLINE 30 DIAS) E DADOS LOCAIS
# =============================================================================
class LicenseManager:
    def __init__(self):
        self.app_dir = os.path.join(os.path.expanduser("~"), ".kronos_pro")
        os.makedirs(self.app_dir, exist_ok=True)
        self.lic_file = os.path.join(self.app_dir, "license_state.json")

    def carregar_estado(self):
        if os.path.exists(self.lic_file):
            try:
                with open(self.lic_file, "r", encoding="utf-8") as f:
                    return json.load(f)
            except Exception:
                return None
        return None

    def guardar_ativacao(self, email):
        dados = {
            "email": email.strip(),
            "ativado_em": time.time(),
            "ultima_validacao": time.time(),
            "maquina": platform.node()
        }
        with open(self.lic_file, "w", encoding="utf-8") as f:
            json.dump(dados, f, ensure_ascii=False, indent=2)
        return dados

    def verificar_validade(self):
        estado = self.carregar_estado()
        if not estado or not estado.get("email"):
            return "nao_ativado", 0, None
        dias_offline = (time.time() - float(estado.get("ultima_validacao", 0))) / 86400.0
        dias_restantes = max(0, int(30 - dias_offline))
        if dias_offline > 30.0:
            return "expirado_offline", 0, estado
        if dias_offline >= 25.0:
            return "aviso_offline", dias_restantes, estado
        return "ativo", dias_restantes, estado


class AtivacaoLicencaDialog(QDialog):
    def __init__(self, lic_mgr, parent=None, aviso_expirado=False):
        super().__init__(parent)
        self.lic_mgr = lic_mgr
        self.setWindowTitle(f"{APP_NAME} — Ativação de Licença")
        self.setFixedSize(430, 320)
        self.setStyleSheet(STYLESHEET)
        self.setWindowIcon(KronosProQtApp.gerar_icone_app_k(128))

        lay = QVBoxLayout(self)
        lay.setContentsMargins(22, 20, 22, 20)
        lay.setSpacing(12)

        lbl_tit = QLabel(f"<span style='color:#38BDF8; font-size:17px; font-weight:bold;'>{APP_NAME}</span><br><span style='color:#94A3B8;'>Ativação e Validação de Licença</span>")
        lay.addWidget(lbl_tit)

        msg = (
            "<span style='color:#F59E0B;'>Passaram mais de 30 dias sem ligação à internet. Ligue-se à rede e confirme a sua conta para renovar o acesso offline.</span>"
            if aviso_expirado else
            "Introduza o e-mail e a palavra-passe fornecidos pelo seu clube ou equipa KRONOS. Após a ativação, a aplicação funciona até <b>30 dias sem internet</b>."
        )
        lbl_desc = QLabel(msg)
        lbl_desc.setWordWrap(True)
        lay.addWidget(lbl_desc)

        form = QFormLayout()
        self.edt_email = QLineEdit()
        self.edt_email.setPlaceholderText("treinador@clube.pt")
        est = self.lic_mgr.carregar_estado()
        if est and est.get("email"):
            self.edt_email.setText(est["email"])
        form.addRow("E-mail:", self.edt_email)

        self.edt_pass = QLineEdit()
        self.edt_pass.setEchoMode(QLineEdit.EchoMode.Password)
        self.edt_pass.setPlaceholderText("••••••••••••")
        form.addRow("Palavra-passe:", self.edt_pass)
        lay.addLayout(form)

        lbl_priv = QLabel("<span style='color:#71717A; font-size:10px;'>Privacidade: Os seus vídeos e análises nunca saem deste computador.</span>")
        lbl_priv.setWordWrap(True)
        lay.addWidget(lbl_priv)

        lay.addStretch()
        h_btn = QHBoxLayout()
        btn_sair = QPushButton("Sair")
        btn_sair.clicked.connect(self.reject)
        h_btn.addWidget(btn_sair)

        btn_entrar = QPushButton("Entrar e Ativar")
        btn_entrar.setObjectName("AccentBtn")
        btn_entrar.clicked.connect(self.validar)
        h_btn.addWidget(btn_entrar)
        lay.addLayout(h_btn)

    def validar(self):
        email = self.edt_email.text().strip()
        pw = self.edt_pass.text().strip()
        if "@" not in email or len(pw) < 4:
            QMessageBox.warning(self, "Dados Inválidos", "Introduza um endereço de e-mail válido e a palavra-passe da sua licença.")
            return
        self.lic_mgr.guardar_ativacao(email)
        self.accept()


# =============================================================================
# LOGÓTIPO VETORIAL NATIVO RETINA
# =============================================================================
class VectorLogoWidget(QWidget):
    def __init__(self):
        super().__init__()
        self.setFixedSize(295, 44)

    def paintEvent(self, event):
        p = QPainter(self)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        p.setRenderHint(QPainter.RenderHint.TextAntialiasing, True)

        ox, oy, tam = 4.0, 4.0, 36.0
        grad = QLinearGradient(ox, oy, ox + tam, oy + tam)
        grad.setColorAt(0.0, QColor("#38BDF8"))
        grad.setColorAt(0.5, QColor("#0284C7"))
        grad.setColorAt(1.0, QColor("#0F172A"))

        escudo = QPolygonF([
            QPointF(ox + tam * 0.50, oy + tam * 0.03),
            QPointF(ox + tam * 0.95, oy + tam * 0.26),
            QPointF(ox + tam * 0.84, oy + tam * 0.95),
            QPointF(ox + tam * 0.50, oy + tam * 0.81),
            QPointF(ox + tam * 0.16, oy + tam * 0.95),
            QPointF(ox + tam * 0.05, oy + tam * 0.26)
        ])
        p.setBrush(QBrush(grad))
        p.setPen(QPen(QColor("#7DD3FC"), 1.6))
        p.drawPolygon(escudo)

        p.setPen(QPen(QColor("#F59E0B"), 2.8, Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
        p.drawLine(QPointF(ox + tam * 0.34, oy + tam * 0.27), QPointF(ox + tam * 0.34, oy + tam * 0.69))

        p.setPen(QPen(QColor("#FFFFFF"), 2.4, Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
        p.drawLine(QPointF(ox + tam * 0.68, oy + tam * 0.27), QPointF(ox + tam * 0.41, oy + tam * 0.48))
        p.drawLine(QPointF(ox + tam * 0.41, oy + tam * 0.48), QPointF(ox + tam * 0.68, oy + tam * 0.69))

        p.setPen(QColor("#FFFFFF"))
        f_title = QFont("Helvetica Neue", 16, QFont.Weight.Bold)
        f_title.setLetterSpacing(QFont.SpacingType.AbsoluteSpacing, 2.0)
        p.setFont(f_title)
        x_kronos = 49.0
        p.drawText(QPointF(x_kronos, 23.0), "KRONOS")

        larg_kronos = p.fontMetrics().horizontalAdvance("KRONOS")
        x_pro = x_kronos + larg_kronos + 8.0
        rect_pro = QRectF(x_pro, 8.0, 42.0, 17.0)
        p.setBrush(QBrush(QColor("#F59E0B")))
        p.setPen(Qt.PenStyle.NoPen)
        p.drawRoundedRect(rect_pro, 4.0, 4.0)

        p.setPen(QColor("#000000"))
        f_pro = QFont(self.font())
        f_pro.setPointSize(9)
        f_pro.setBold(True)
        p.setFont(f_pro)
        p.drawText(rect_pro, Qt.AlignmentFlag.AlignCenter, "PRO")

        p.setPen(QColor("#38BDF8"))
        f_sub = QFont("Helvetica Neue", 7, QFont.Weight.Bold)
        f_sub.setLetterSpacing(QFont.SpacingType.AbsoluteSpacing, 0.8)
        p.setFont(f_sub)
        p.drawText(QPointF(50.0, 36.5), APP_SUBTITLE.upper())


# =============================================================================
# BOTÕES DE TRANSPORTE VETORIAIS
# =============================================================================
class VectorTransportButton(QPushButton):
    def __init__(self, modo="play", parent=None):
        super().__init__(parent)
        self.modo = modo
        self.hovered = False
        if modo in ("play", "pause"):
            self.setFixedSize(48, 32)
        else:
            self.setFixedSize(76, 32)
        self.setCursor(Qt.CursorShape.PointingHandCursor)

    def definir_modo(self, novo_modo):
        self.modo = novo_modo
        self.update()

    def enterEvent(self, event):
        self.hovered = True
        self.update()
        super().enterEvent(event)

    def leaveEvent(self, event):
        self.hovered = False
        self.update()
        super().leaveEvent(event)

    def paintEvent(self, event):
        p = QPainter(self)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        p.setRenderHint(QPainter.RenderHint.TextAntialiasing, True)
        w, h = float(self.width()), float(self.height())

        if self.modo in ("play", "pause"):
            cor_bg = QColor("#FBBF24") if self.hovered else QColor("#F59E0B")
            p.setBrush(QBrush(cor_bg))
            p.setPen(QPen(QColor("#FDE047"), 1.0))
            p.drawRoundedRect(QRectF(1.0, 1.0, w - 2.0, h - 2.0), 6.0, 6.0)

            p.setBrush(QBrush(QColor("#000000")))
            p.setPen(Qt.PenStyle.NoPen)
            cx, cy = w / 2.0, h / 2.0

            if self.modo == "play":
                tri = QPolygonF([
                    QPointF(cx - 5.0, cy - 7.5),
                    QPointF(cx - 5.0, cy + 7.5),
                    QPointF(cx + 8.0, cy)
                ])
                p.drawPolygon(tri)
            else:
                p.drawRoundedRect(QRectF(cx - 6.5, cy - 7.0, 4.5, 14.0), 1.2, 1.2)
                p.drawRoundedRect(QRectF(cx + 2.0, cy - 7.0, 4.5, 14.0), 1.2, 1.2)
        else:
            cor_bg = QColor("#1A1A28") if self.hovered else QColor("#101018")
            cor_borda = QColor("#38BDF8") if self.hovered else QColor("#262638")
            p.setBrush(QBrush(cor_bg))
            p.setPen(QPen(cor_borda, 1.2))
            p.drawRoundedRect(QRectF(1.0, 1.0, w - 2.0, h - 2.0), 5.0, 5.0)

            p.setBrush(QBrush(QColor("#38BDF8") if self.hovered else QColor("#E2E8F0")))
            p.setPen(Qt.PenStyle.NoPen)
            cy = h / 2.0

            if self.modo == "back15":
                t1 = QPolygonF([QPointF(16.0, cy - 5.5), QPointF(16.0, cy + 5.5), QPointF(9.0, cy)])
                t2 = QPolygonF([QPointF(23.5, cy - 5.5), QPointF(23.5, cy + 5.5), QPointF(16.5, cy)])
                p.drawPolygon(t1)
                p.drawPolygon(t2)
                p.setPen(QColor("#FFFFFF"))
                p.setFont(QFont("SF Mono", 10, QFont.Weight.Bold))
                p.drawText(QRectF(26.0, 0.0, w - 30.0, h), Qt.AlignmentFlag.AlignCenter, "15s")
            elif self.modo == "fwd15":
                p.setPen(QColor("#FFFFFF"))
                p.setFont(QFont("SF Mono", 10, QFont.Weight.Bold))
                p.drawText(QRectF(6.0, 0.0, w - 30.0, h), Qt.AlignmentFlag.AlignCenter, "15s")
                p.setPen(Qt.PenStyle.NoPen)
                t1 = QPolygonF([QPointF(w - 23.5, cy - 5.5), QPointF(w - 23.5, cy + 5.5), QPointF(w - 16.5, cy)])
                t2 = QPolygonF([QPointF(w - 16.0, cy - 5.5), QPointF(w - 16.0, cy + 5.5), QPointF(w - 9.0, cy)])
                p.drawPolygon(t1)
                p.drawPolygon(t2)


class TacticalDatabase:
    def __init__(self):
        self.conn = sqlite3.connect(":memory:", check_same_thread=False)
        self.cursor = self.conn.cursor()
        self.cursor.execute("""
            CREATE TABLE eventos (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                acao TEXT,
                inicio REAL,
                fim REAL,
                momento REAL,
                zona TEXT,
                cx REAL,
                cy REAL,
                labels TEXT,
                nota TEXT
            )
        """)
        self.cursor.execute("CREATE INDEX idx_acao ON eventos(acao)")
        self.conn.commit()

    def sincronizar_lista(self, lista_eventos):
        self.cursor.execute("DELETE FROM eventos")
        for ev in lista_eventos:
            cx, cy = ev.get("coord", (0.5, 0.5))
            lbl_str = ",".join(ev.get("labels", []))
            self.cursor.execute(
                "INSERT INTO eventos (acao, inicio, fim, momento, zona, cx, cy, labels, nota) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)",
                (ev["acao"], ev["inicio"], ev["fim"], ev["momento"], ev.get("zona", ""), cx, cy, lbl_str, ev.get("nota", ""))
            )
        self.conn.commit()

    def contagem_por_acao(self):
        self.cursor.execute("SELECT acao, COUNT(*) FROM eventos GROUP BY acao")
        return dict(self.cursor.fetchall())


class VideoOptimizeWorker(QThread):
    progresso = pyqtSignal(str)
    concluido = pyqtSignal(str)
    erro = pyqtSignal(str)

    def __init__(self, caminho_origem):
        super().__init__()
        self.caminho_origem = caminho_origem
        self.ffmpeg_bin = shutil.which("ffmpeg")

    def run(self):
        if not self.ffmpeg_bin:
            self.erro.emit("FFmpeg não encontrado no sistema para otimização.")
            return
        try:
            base, _ = os.path.splitext(self.caminho_origem)
            destino = f"{base}_kronos_otimizado.mp4"
            self.progresso.emit("A converter vídeo para MP4 (H.264) otimizado...")
            cmd = [
                self.ffmpeg_bin, "-y", "-i", self.caminho_origem,
                "-c:v", "libx264", "-preset", "veryfast", "-crf", "20",
                "-pix_fmt", "yuv420p", "-c:a", "aac", "-b:a", "128k",
                "-movflags", "+faststart", destino
            ]
            res = subprocess.run(cmd, stdout=subprocess.DEVNULL, stderr=subprocess.PIPE)
            if res.returncode == 0 and os.path.exists(destino):
                self.concluido.emit(destino)
            else:
                self.erro.emit("Falha ao otimizar o ficheiro de vídeo.")
        except Exception as e:
            self.erro.emit(str(e))


class SubjanelaCortesGrupoDialog(QDialog):
    def __init__(self, app_ref, nome_grupo, cor_hex):
        super().__init__(app_ref)
        self.app = app_ref
        self.nome_grupo = nome_grupo
        self.setWindowTitle(f"Momentos Registados — {nome_grupo}")
        self.resize(520, 380)

        v_lay = QVBoxLayout(self)
        v_lay.setContentsMargins(12, 12, 12, 12)
        v_lay.setSpacing(8)

        cab = QLabel(
            f"<span style='color:{cor_hex}; font-size:15px; font-weight:bold;'>● {nome_grupo.upper()}</span><br>"
            f"<span style='color:#94A3B8; font-size:11px;'>Clique em qualquer momento abaixo para saltar diretamente para o início desse corte:</span>"
        )
        v_lay.addWidget(cab)

        self.lista_w = QListWidget()
        v_lay.addWidget(self.lista_w, 1)

        self.cortes_filtrados = [
            (idx_global, ev) for idx_global, ev in enumerate(self.app.eventos_registados)
            if ev["acao"] == nome_grupo
        ]

        if not self.cortes_filtrados:
            item_vazio = QListWidgetItem("Ainda não existem cortes registados para este grupo.")
            item_vazio.setFlags(Qt.ItemFlag.NoItemFlags)
            self.lista_w.addItem(item_vazio)
        else:
            for num, (idx_global, ev) in enumerate(self.cortes_filtrados, start=1):
                t_ini_str = self.app.formatar_tempo(ev["inicio"], True)
                t_fim_str = self.app.formatar_tempo(ev["fim"], True)
                dur = round(ev["fim"] - ev["inicio"], 1)
                zona = ev.get("zona", "")
                etiquetas = ", ".join(ev.get("labels", []))
                nota = ev.get("nota", "")

                texto_item = f"Corte #{num}   |   Início: {t_ini_str}  →  Fim: {t_fim_str}  ({dur}s)"
                if zona:
                    texto_item += f"\n   Zona: {zona}"
                if etiquetas:
                    texto_item += f"  •  Etiquetas: {etiquetas}"
                if nota:
                    texto_item += f"  •  Nota: {nota}"

                it = QListWidgetItem(texto_item)
                it.setData(Qt.ItemDataRole.UserRole, idx_global)
                self.lista_w.addItem(it)

        self.lista_w.itemClicked.connect(self.ao_clicar_corte_lista)

        h_btns = QHBoxLayout()
        btn_play_all = QPushButton("Reproduzir Sequência Completa do Grupo")
        btn_play_all.setObjectName("AccentBtn")
        btn_play_all.setEnabled(len(self.cortes_filtrados) > 0)
        btn_play_all.clicked.connect(self.reproduzir_todos)
        h_btns.addWidget(btn_play_all)

        btn_send_studio = QPushButton("Enviar Grupo p/ Vision Studio")
        btn_send_studio.setEnabled(len(self.cortes_filtrados) > 0)
        btn_send_studio.clicked.connect(self.enviar_grupo_estudio)
        h_btns.addWidget(btn_send_studio)

        btn_fechar = QPushButton("Fechar")
        btn_fechar.clicked.connect(self.accept)
        h_btns.addWidget(btn_fechar)
        v_lay.addLayout(h_btns)

    def ao_clicar_corte_lista(self, item):
        idx_global = item.data(Qt.ItemDataRole.UserRole)
        if idx_global is None:
            return
        ev = self.app.eventos_registados[idx_global]
        self.app.bloco_selecionado = idx_global
        self.app.entry_nota.setText(ev.get("nota", ""))
        self.app.playlist_ativa = [ev]
        self.app.idx_playlist = 0
        self.app.ir_para_tempo(ev["inicio"])
        self.app.media_player.play()
        self.app.btn_play.definir_modo("pause")
        self.app.timeline_widget.update()

    def reproduzir_todos(self):
        lista = [ev for _, ev in self.cortes_filtrados]
        self.app.iniciar_playlist_cortes(lista, self)

    def enviar_grupo_estudio(self):
        lista = [ev for _, ev in self.cortes_filtrados]
        self.app.importar_lista_especifica_para_estudio(lista)
        self.accept()


class ConfigurarModoCorteBotaoDialog(QDialog):
    OPCOES_INDIVIDUAIS = [
        ("Ao Clique (Inicia ou Termina no Clique)", "clique"),
        ("0 segundos (Exato)", 0.0),
        ("2 segundos", 2.0),
        ("3 segundos", 3.0),
        ("5 segundos", 5.0),
        ("8 segundos", 8.0),
        ("10 segundos", 10.0),
        ("15 segundos", 15.0),
        ("20 segundos", 20.0),
        ("30 segundos", 30.0),
        ("Personalizável (Definir valor)...", "custom")
    ]

    def __init__(self, parent, item_dict, lista_momentos_nomes):
        super().__init__(parent)
        self.item = item_dict
        self.setWindowTitle(f"Configurar Ativação e Corte — {item_dict['nome']}")
        self.resize(450, 490)

        lay = QVBoxLayout(self)
        form = QFormLayout()

        lbl_sec1 = QLabel("<b style='color:#38BDF8; font-size:13px;'>1. MODO DE ATIVAÇÃO DO CORTE (PRÉ E PÓS)</b>")
        lay.addWidget(lbl_sec1)

        self.cb_preset_rapido = QComboBox()
        self.cb_preset_rapido.addItems([
            "-- Escolher Predefinição Rápida --",
            "Ao Clique → Até Clicar Novamente (In / Out Manual)",
            "10s Antes e 10s Depois da Tecla/Clique",
            "5s Antes e 5s Depois da Tecla/Clique",
            "3s Antes e 3s Depois da Tecla/Clique",
            "15s Antes e 15s Depois da Tecla/Clique",
            "Começar Ao Clique → Terminar 10s Depois",
            "Começar 10s Antes → Terminar Ao Clique"
        ])
        self.cb_preset_rapido.currentIndexChanged.connect(self.ao_escolher_preset_rapido)
        form.addRow("Predefinições Rápidas:", self.cb_preset_rapido)

        self.cb_pre = QComboBox()
        for rot, val in self.OPCOES_INDIVIDUAIS:
            self.cb_pre.addItem(rot, val)
        self.cb_pre.currentIndexChanged.connect(self.atualizar_visibilidade_spins)
        form.addRow("Início (Pré-Corte):", self.cb_pre)

        self.sp_pre = QDoubleSpinBox()
        self.sp_pre.setRange(0.0, 180.0)
        self.sp_pre.setSingleStep(1.0)
        self.sp_pre.setSuffix(" segundos antes")
        self.sp_pre.setValue(float(item_dict.get("pre_val", 10.0)))
        form.addRow("Valor Pré Personalizado:", self.sp_pre)

        self.cb_pos = QComboBox()
        for rot, val in self.OPCOES_INDIVIDUAIS:
            self.cb_pos.addItem(rot, val)
        self.cb_pos.currentIndexChanged.connect(self.atualizar_visibilidade_spins)
        form.addRow("Fim (Pós-Corte):", self.cb_pos)

        self.sp_pos = QDoubleSpinBox()
        self.sp_pos.setRange(0.0, 180.0)
        self.sp_pos.setSingleStep(1.0)
        self.sp_pos.setSuffix(" segundos depois")
        self.sp_pos.setValue(float(item_dict.get("pos_val", 10.0)))
        form.addRow("Valor Pós Personalizado:", self.sp_pos)

        self.selecionar_valor_combo(self.cb_pre, item_dict.get("pre_tipo", "clique" if item_dict["tipo"] == "momento" else "segundos"), item_dict.get("pre_val", 10.0))
        self.selecionar_valor_combo(self.cb_pos, item_dict.get("pos_tipo", "clique" if item_dict["tipo"] == "momento" else "segundos"), item_dict.get("pos_val", 10.0))

        form.addRow(QLabel("<b style='color:#38BDF8; font-size:13px; margin-top:8px;'>2. APARÊNCIA E TECLA DE ATALHO</b>"))

        self.edt_nome = QLineEdit(item_dict["nome"])
        form.addRow("Nome do Botão:", self.edt_nome)

        self.edt_atalho = QLineEdit(item_dict.get("atalho", ""))
        self.edt_atalho.setMaxLength(2)
        form.addRow("Tecla de Atalho:", self.edt_atalho)

        self.cor_escolhida = item_dict.get("cor", "#0284C7")
        self.btn_cor = QPushButton(self.cor_escolhida)
        self.btn_cor.setStyleSheet(f"background-color: {self.cor_escolhida}; color: white; border: 1px solid white;")
        self.btn_cor.clicked.connect(self.escolher_cor)
        form.addRow("Cor do Botão e Corte:", self.btn_cor)

        self.sp_fonte = QSpinBox()
        self.sp_fonte.setRange(8, 26)
        self.sp_fonte.setValue(item_dict.get("fonte", 11))
        form.addRow("Tamanho da Letra (pt):", self.sp_fonte)

        if item_dict["tipo"] == "acao":
            self.cb_link = QComboBox()
            self.cb_link.addItems(["(Nenhum)"] + lista_momentos_nomes)
            if item_dict.get("link_momento") in lista_momentos_nomes:
                self.cb_link.setCurrentText(item_dict["link_momento"])
            form.addRow("Ativar Momento Automático:", self.cb_link)

        lay.addLayout(form)
        self.atualizar_visibilidade_spins()

        h_btns = QHBoxLayout()
        btn_cancel = QPushButton("Cancelar")
        btn_cancel.clicked.connect(self.reject)
        h_btns.addWidget(btn_cancel)

        btn_ok = QPushButton("Guardar Configuração do Botão")
        btn_ok.setObjectName("SuccessBtn")
        btn_ok.clicked.connect(self.guardar)
        h_btns.addWidget(btn_ok)
        lay.addLayout(h_btns)

    def selecionar_valor_combo(self, combo, tipo, val):
        if tipo == "clique":
            combo.setCurrentIndex(0)
            return
        for i in range(1, combo.count() - 1):
            if combo.itemData(i) == float(val):
                combo.setCurrentIndex(i)
                return
        combo.setCurrentIndex(combo.count() - 1)

    def atualizar_visibilidade_spins(self):
        self.sp_pre.setVisible(self.cb_pre.currentData() == "custom")
        self.sp_pos.setVisible(self.cb_pos.currentData() == "custom")

    def ao_escolher_preset_rapido(self, idx):
        mapa = {
            1: ("clique", 0.0, "clique", 0.0),
            2: ("segundos", 10.0, "segundos", 10.0),
            3: ("segundos", 5.0, "segundos", 5.0),
            4: ("segundos", 3.0, "segundos", 3.0),
            5: ("segundos", 15.0, "segundos", 15.0),
            6: ("clique", 0.0, "segundos", 10.0),
            7: ("segundos", 10.0, "clique", 0.0),
        }
        if idx in mapa:
            t_pre, v_pre, t_pos, v_pos = mapa[idx]
            self.selecionar_valor_combo(self.cb_pre, t_pre, v_pre)
            self.selecionar_valor_combo(self.cb_pos, t_pos, v_pos)
            self.atualizar_visibilidade_spins()

    def escolher_cor(self):
        c = QColorDialog.getColor(QColor(self.cor_escolhida), self, "Escolher Cor")
        if c.isValid():
            self.cor_escolhida = c.name().upper()
            self.btn_cor.setText(self.cor_escolhida)
            self.btn_cor.setStyleSheet(f"background-color: {self.cor_escolhida}; color: white; border: 1px solid white;")

    def guardar(self):
        self.item["nome"] = self.edt_nome.text().strip() or self.item["nome"]
        self.item["atalho"] = self.edt_atalho.text().strip().lower()
        self.item["cor"] = self.cor_escolhida
        self.item["fonte"] = self.sp_fonte.value()

        d_pre = self.cb_pre.currentData()
        if d_pre == "clique":
            self.item["pre_tipo"], self.item["pre_val"] = "clique", 0.0
        elif d_pre == "custom":
            self.item["pre_tipo"], self.item["pre_val"] = "segundos", float(self.sp_pre.value())
        else:
            self.item["pre_tipo"], self.item["pre_val"] = "segundos", float(d_pre)

        d_pos = self.cb_pos.currentData()
        if d_pos == "clique":
            self.item["pos_tipo"], self.item["pos_val"] = "clique", 0.0
        elif d_pos == "custom":
            self.item["pos_tipo"], self.item["pos_val"] = "segundos", float(self.sp_pos.value())
        else:
            self.item["pos_tipo"], self.item["pos_val"] = "segundos", float(d_pos)

        if self.item["tipo"] == "acao":
            lk = self.cb_link.currentText()
            self.item["link_momento"] = lk if lk != "(Nenhum)" else ""

        self.accept()


class DraggableCodeWidget(QWidget):
    MARGEM_BORDA = 7

    def __init__(self, canvas_parent, item_dict, app_ref):
        super().__init__(canvas_parent)
        self.canvas = canvas_parent
        self.item = item_dict
        self.app = app_ref

        self.setGeometry(item_dict["x"], item_dict["y"], item_dict["w"], item_dict["h"])
        self.modo_arrasto = None
        self.pos_inicial_rato = QPoint()
        self.geom_inicial = QRect()
        self.setMouseTracking(True)

    def _detetar_zona_redimensionamento(self, pos):
        x, y, w, h = pos.x(), pos.y(), self.width(), self.height()
        esq = x <= self.MARGEM_BORDA
        dir_ = x >= w - self.MARGEM_BORDA
        cima = y <= self.MARGEM_BORDA
        baixo = y >= h - self.MARGEM_BORDA

        if cima and esq:
            return "top_left"
        if cima and dir_:
            return "top_right"
        if baixo and esq:
            return "bottom_left"
        if baixo and dir_:
            return "bottom_right"
        if esq:
            return "left"
        if dir_:
            return "right"
        if cima:
            return "top"
        if baixo:
            return "bottom"
        return "move"

    def _cursor_para_zona(self, zona):
        mapa = {
            "left": Qt.CursorShape.SizeHorCursor,
            "right": Qt.CursorShape.SizeHorCursor,
            "top": Qt.CursorShape.SizeVerCursor,
            "bottom": Qt.CursorShape.SizeVerCursor,
            "top_left": Qt.CursorShape.SizeFDiagCursor,
            "bottom_right": Qt.CursorShape.SizeFDiagCursor,
            "top_right": Qt.CursorShape.SizeBDiagCursor,
            "bottom_left": Qt.CursorShape.SizeBDiagCursor,
        }
        return mapa.get(zona, Qt.CursorShape.SizeAllCursor)

    def _calcular_tamanho_fonte_dinamico(self):
        base_f = float(self.item.get("fonte", 11))
        fator_alt = max(0.65, min(1.45, self.height() / 36.0))
        fator_larg = max(0.70, min(1.25, self.width() / 140.0))
        return max(7, min(28, int(round(base_f * min(fator_alt, fator_larg)))))

    def obter_resumo_modo_corte(self):
        if self.item["tipo"] not in ("momento", "acao"):
            return ""
        pre_t = self.item.get("pre_tipo", "clique" if self.item["tipo"] == "momento" else "segundos")
        pos_t = self.item.get("pos_tipo", "clique" if self.item["tipo"] == "momento" else "segundos")
        pre_v = int(self.item.get("pre_val", 10))
        pos_v = int(self.item.get("pos_val", 10))
        if pre_t == "clique" and pos_t == "clique":
            return "CLIQUE IN/OUT"
        s_pre = "CLIQUE" if pre_t == "clique" else f"-{pre_v}s"
        s_pos = "CLIQUE" if pos_t == "clique" else f"+{pos_v}s"
        return f"{s_pre} / {s_pos}"

    def paintEvent(self, event):
        p = QPainter(self)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        p.setRenderHint(QPainter.RenderHint.TextAntialiasing, True)
        w, h = float(self.width()), float(self.height())
        tipo = self.item["tipo"]

        if tipo == "minicampo":
            mod = getattr(self.app, "modalidade_atual", "futebol")
            margem_bal = 7.0 if mod == "futebol" else 1.0
            self.app._desenhar_campo_por_modalidade(
                p, mod, QRectF(margem_bal, 1.0, w - 2.0 * margem_bal, h - 2.0), mostrar_zonas=True
            )

            px = self.app.coord_campo_atual[0] * w
            py = self.app.coord_campo_atual[1] * h
            p.setBrush(QBrush(QColor("#F59E0B")))
            p.setPen(QPen(Qt.GlobalColor.black, 1.5))
            p.drawEllipse(QPointF(px, py), 6.0, 6.0)

            p.setPen(QColor("#FDE047"))
            p.setFont(QFont("Helvetica Neue", max(7, int(min(11, h * 0.07))), QFont.Weight.Bold))
            p.drawText(QPointF(6.0, h - 5.0), self.app.zona_campo_atual)
        else:
            cor_base = QColor(self.item.get("cor", "#0284C7"))
            nome = self.item["nome"]
            ativo = (
                (tipo == "momento" and self.app.momento_ativo == nome)
                or (nome in self.app.acoes_em_gravacao)
                or (tipo == "label" and nome in self.app.labels_ativas)
            )
            if tipo == "label" and ativo:
                cor_base = QColor("#0284C7")

            raio = float(self.item.get("raio", 6))
            grad_btn = QLinearGradient(0, 0, 0, h)
            grad_btn.setColorAt(0.0, cor_base.lighter(115))
            grad_btn.setColorAt(1.0, cor_base.darker(115))
            p.setBrush(QBrush(grad_btn))

            selecionado = (getattr(self.canvas, "widget_selecionado", None) is self)
            if ativo:
                p.setPen(QPen(QColor("#FDE047"), 2.5))
            elif selecionado and self.canvas.modo_edicao:
                p.setPen(QPen(QColor("#F97316"), 2.5))
            else:
                p.setPen(QPen(QColor(255, 255, 255, 65), 1.0))
            p.drawRoundedRect(QRectF(1.0, 1.0, w - 2.0, h - 2.0), raio, raio)

            if tipo == "momento":
                p.fillRect(QRectF(8.0, 3.0, w - 16.0, 2.0), QColor(255, 255, 255, 150))

            p.setPen(QColor("#FFFFFF"))
            tam_f = self._calcular_tamanho_fonte_dinamico()
            fnt = QFont("Helvetica Neue", tam_f, QFont.Weight.Bold)
            p.setFont(fnt)

            txt = nome
            if ativo and tipo in ("momento", "acao"):
                txt = f"REC • {txt}"
            if self.item.get("atalho"):
                txt += f" [{self.item['atalho'].upper()}]"

            if tipo in ("momento", "acao") and h >= 32:
                p.drawText(QRectF(4.0, 1.0, w - 8.0, h - 11.0), Qt.AlignmentFlag.AlignCenter | Qt.TextFlag.TextWordWrap, txt)
                p.setPen(QColor(255, 255, 255, 175))
                p.setFont(QFont("SF Mono", max(6, int(tam_f * 0.65)), QFont.Weight.Bold))
                p.drawText(QRectF(4.0, h - 12.0, w - 8.0, 11.0), Qt.AlignmentFlag.AlignCenter, self.obter_resumo_modo_corte())
            else:
                p.drawText(QRectF(4.0, 2.0, w - 8.0, h - 4.0), Qt.AlignmentFlag.AlignCenter | Qt.TextFlag.TextWordWrap, txt)

        if self.canvas.modo_edicao:
            selecionado = (getattr(self.canvas, "widget_selecionado", None) is self)
            p.setBrush(Qt.BrushStyle.NoBrush)
            p.setPen(QPen(QColor("#F97316" if selecionado else "#38BDF8"), 1.8 if selecionado else 1.5, Qt.PenStyle.DashLine))
            p.drawRect(QRectF(0.5, 0.5, w - 1.0, h - 1.0))

            p.setBrush(QBrush(QColor("#38BDF8")))
            p.setPen(Qt.PenStyle.NoPen)
            m = 6.0
            for px_m, py_m in [
                (0.0, 0.0), (w - m, 0.0), (0.0, h - m), (w - m, h - m),
                (w / 2.0 - m / 2.0, 0.0), (w / 2.0 - m / 2.0, h - m),
                (0.0, h / 2.0 - m / 2.0), (w - m, h / 2.0 - m / 2.0)
            ]:
                p.drawRect(QRectF(px_m, py_m, m, m))

    def contextMenuEvent(self, event):
        if self.item["tipo"] != "minicampo":
            self.abrir_menu_contexto(event.globalPos())

    def mousePressEvent(self, event):
        if event.button() == Qt.MouseButton.LeftButton:
            self.canvas.selecionar_widget(self)
            if self.canvas.modo_edicao:
                self.app.registar_historico_codigo()
                self.modo_arrasto = self._detetar_zona_redimensionamento(event.position().toPoint())
                self.pos_inicial_rato = event.globalPosition().toPoint()
                self.geom_inicial = self.geometry()
                self.raise_()
                self.update()
                event.accept()
                return

            tipo = self.item["tipo"]
            if tipo == "minicampo":
                nx = max(0.0, min(1.0, event.position().x() / max(1, self.width())))
                ny = max(0.0, min(1.0, event.position().y() / max(1, self.height())))
                mod = getattr(self.app, "modalidade_atual", "futebol")
                if mod == "voleibol":
                    zona_v, corr_v = self.app._calcular_zona_voleibol(nx, ny)
                    self.app.ao_selecionar_zona_campo(zona_v, (round(nx, 3), round(ny, 3)), corr_v)
                elif mod == "basquetebol":
                    zona_b, corr_b = self.app._calcular_zona_basquetebol(nx, ny)
                    self.app.ao_selecionar_zona_campo(zona_b, (round(nx, 3), round(ny, 3)), corr_b)
                elif mod == "andebol":
                    zona_a, corr_a = self.app._calcular_zona_andebol(nx, ny)
                    self.app.ao_selecionar_zona_campo(zona_a, (round(nx, 3), round(ny, 3)), corr_a)
                else:
                    col = min(5, int(nx * 6))
                    row = min(2, int(ny * 3))
                    num_z = col * 3 + row + 1
                    corredores = ["Corredor Esq", "Corredor Central", "Corredor Dir"]
                    self.app.ao_selecionar_zona_campo(f"Zona {num_z} ({corredores[row]})", (round(nx, 3), round(ny, 3)), corredores[row])
                self.update()
            elif tipo in ("momento", "acao"):
                self.app.acionar_item_codificacao(self.item)
            elif tipo == "label":
                self.app.toggle_label(self.item["nome"])
            event.accept()
            return
        super().mousePressEvent(event)

    def mouseMoveEvent(self, event):
        if not self.canvas.modo_edicao:
            self.setCursor(Qt.CursorShape.PointingHandCursor)
            return

        if not self.modo_arrasto:
            zona_hover = self._detetar_zona_redimensionamento(event.position().toPoint())
            self.setCursor(self._cursor_para_zona(zona_hover))
            return

        delta = event.globalPosition().toPoint() - self.pos_inicial_rato
        dx, dy = delta.x(), delta.y()
        min_w = 110 if self.item["tipo"] == "minicampo" else 48
        min_h = 70 if self.item["tipo"] == "minicampo" else 24

        g = self.geom_inicial
        x, y, w, h = g.x(), g.y(), g.width(), g.height()

        if self.modo_arrasto == "move":
            x = max(0, x + dx)
            y = max(0, y + dy)
        else:
            if "right" in self.modo_arrasto:
                w = max(min_w, w + dx)
            if "bottom" in self.modo_arrasto:
                h = max(min_h, h + dy)
            if "left" in self.modo_arrasto:
                novo_w = max(min_w, w - dx)
                x = max(0, x + (w - novo_w))
                w = novo_w
            if "top" in self.modo_arrasto:
                novo_h = max(min_h, h - dy)
                y = max(0, y + (h - novo_h))
                h = novo_h

        self.setGeometry(x, y, w, h)
        self.item["x"], self.item["y"], self.item["w"], self.item["h"] = x, y, w, h
        self.canvas.ajustar_tamanho_minimo_tela()
        self.update()

    def mouseReleaseEvent(self, event):
        self.modo_arrasto = None

    def abrir_menu_contexto(self, global_pos):
        menu = QMenu(self)
        nome = self.item["nome"]

        if self.item["tipo"] in ("momento", "acao"):
            titulo_m = QAction(f"MODO DE CORTE: {nome.upper()}", self)
            titulo_m.setEnabled(False)
            menu.addAction(titulo_m)

            presets = [
                ("Ao Clique → Até Clicar Novamente (In / Out Manual)", "clique", 0.0, "clique", 0.0),
                ("10s Antes  /  10s Depois (Padrão)", "segundos", 10.0, "segundos", 10.0),
                ("5s Antes  /  5s Depois (Rápido)", "segundos", 5.0, "segundos", 5.0),
                ("3s Antes  /  3s Depois (Curto)", "segundos", 3.0, "segundos", 3.0),
                ("15s Antes  /  15s Depois (Alargado)", "segundos", 15.0, "segundos", 15.0),
                ("20s Antes  /  20s Depois", "segundos", 20.0, "segundos", 20.0),
                ("Início Ao Clique  /  Fim 10s Depois", "clique", 0.0, "segundos", 10.0),
                ("Início 10s Antes  /  Fim Ao Clique", "segundos", 10.0, "clique", 0.0),
            ]
            for rot, pt, pv, pst, psv in presets:
                act = QAction(rot, self)
                act.triggered.connect(lambda checked, a=pt, b=pv, c=pst, d=psv: self.aplicar_preset_direto(a, b, c, d))
                menu.addAction(act)

            menu.addSeparator()

        act_edit = QAction("Personalizar Pré/Pós Independentes, Cor e Tecla...", self)
        act_edit.triggered.connect(self.editar_propriedades)
        menu.addAction(act_edit)

        if self.canvas.modo_edicao:
            menu.addSeparator()
            act_dup = QAction("Duplicar Botão", self)
            act_dup.triggered.connect(lambda: self.canvas.duplicar_item(self.item))
            menu.addAction(act_dup)

            act_del = QAction("Eliminar Botão da Tela", self)
            act_del.triggered.connect(lambda: self.canvas.eliminar_item(self.item))
            menu.addAction(act_del)

        menu.exec(global_pos)

    def aplicar_preset_direto(self, pre_t, pre_v, pos_t, pos_v):
        self.item["pre_tipo"] = pre_t
        self.item["pre_val"] = pre_v
        self.item["pos_tipo"] = pos_t
        self.item["pos_val"] = pos_v
        self.update()

    def editar_propriedades(self):
        momentos_nomes = [i["nome"] for i in self.app.layout_items if i["tipo"] == "momento"]
        dlg = ConfigurarModoCorteBotaoDialog(self.app, self.item, momentos_nomes)
        if dlg.exec():
            self.setGeometry(self.item["x"], self.item["y"], self.item["w"], self.item["h"])
            self.update()
            self.app.sincronizar_estruturas_desde_canvas()


class FreeformCodeCanvas(QWidget):
    def __init__(self, app_ref):
        super().__init__()
        self.app = app_ref
        self.modo_edicao = False
        self.widgets_itens = []
        self.widget_selecionado = None
        self.setMinimumSize(290, 640)

    def paintEvent(self, event):
        p = QPainter(self)
        p.fillRect(0, 0, self.width(), self.height(), QColor("#000000"))

        if self.modo_edicao:
            p.setPen(QPen(QColor("#1E293B"), 1.5))
            passo = 16
            for x in range(8, self.width(), passo):
                for y in range(8, self.height(), passo):
                    p.drawPoint(x, y)

    def selecionar_widget(self, w_alvo):
        self.widget_selecionado = w_alvo
        self.atualizar_visual_todos()

    def mover_selecionado_teclado(self, dx, dy):
        w = self.widget_selecionado
        if w is None or w not in self.widgets_itens:
            return
        self.app.registar_historico_codigo()
        nx = max(0, w.x() + dx)
        ny = max(0, w.y() + dy)
        w.move(nx, ny)
        w.item["x"], w.item["y"] = nx, ny
        self.ajustar_tamanho_minimo_tela()
        w.update()

    def apagar_selecionado_teclado(self):
        w = self.widget_selecionado
        if w is None or w not in self.widgets_itens:
            return
        item_alvo = w.item
        self.widget_selecionado = None
        self.eliminar_item(item_alvo)

    def reconstruir_widgets(self):
        for w in self.widgets_itens:
            w.setParent(None)
            w.deleteLater()
        self.widgets_itens.clear()
        self.widget_selecionado = None

        for item in self.app.layout_items:
            if getattr(self.app, "campo_oculto", False) and item.get("tipo") == "minicampo":
                continue
            w = DraggableCodeWidget(self, item, self.app)
            w.show()
            self.widgets_itens.append(w)

        self.ajustar_tamanho_minimo_tela()
        self.update()

    def atualizar_visual_todos(self):
        for w in self.widgets_itens:
            w.update()
        self.update()

    def ajustar_tamanho_minimo_tela(self):
        max_x = 280
        max_y = 600
        for it in self.app.layout_items:
            if getattr(self.app, "campo_oculto", False) and it.get("tipo") == "minicampo":
                continue
            max_x = max(max_x, it["x"] + it["w"] + 20)
            max_y = max(max_y, it["y"] + it["h"] + 30)
        self.setMinimumSize(max_x, max_y)

    def adicionar_novo_item(self, tipo):
        self.app.registar_historico_codigo()
        scroll_h = self.app.scroll_canvas.verticalScrollBar().value() if hasattr(self.app, "scroll_canvas") else 0
        x_livre = 20 + (len(self.app.layout_items) * 7) % 120
        y_livre = scroll_h + 24 + (len(self.app.layout_items) * 11) % 180

        if tipo == "momento":
            novo = {"tipo": "momento", "nome": "Novo Momento", "atalho": "6", "cor": "#0D9488", "x": x_livre, "y": y_livre, "w": 256, "h": 36, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0}
        elif tipo == "acao":
            novo = {"tipo": "acao", "nome": "Nova Ação", "atalho": "z", "cor": "#E11D48", "x": x_livre, "y": y_livre, "w": 124, "h": 44, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 10.0, "pos_tipo": "segundos", "pos_val": 10.0}
        else:
            novo = {"tipo": "label", "nome": "Nova Etiqueta", "atalho": "", "cor": "#14141D", "x": x_livre, "y": y_livre, "w": 82, "h": 28, "fonte": 10, "raio": 4}

        self.app.layout_items.append(novo)
        self.reconstruir_widgets()
        if self.widgets_itens:
            self.selecionar_widget(self.widgets_itens[-1])
        self.app.sincronizar_estruturas_desde_canvas()

    def duplicar_item(self, item_orig):
        self.app.registar_historico_codigo()
        copia = dict(item_orig)
        copia["nome"] = item_orig["nome"] + " (2)"
        copia["x"] = item_orig["x"] + 16
        copia["y"] = item_orig["y"] + 16
        self.app.layout_items.append(copia)
        self.reconstruir_widgets()
        self.app.sincronizar_estruturas_desde_canvas()

    def eliminar_item(self, item_alvo):
        if item_alvo in self.app.layout_items:
            self.app.registar_historico_codigo()
            self.app.layout_items.remove(item_alvo)
            self.reconstruir_widgets()
            self.app.sincronizar_estruturas_desde_canvas()

    def descartar_campo(self):
        self.app.registar_historico_codigo()
        self.app.campo_oculto = True
        self.app.layout_items = [it for it in self.app.layout_items if it.get("tipo") != "minicampo"]
        self.reconstruir_widgets()
        self.app.sincronizar_estruturas_desde_canvas()

    def repor_campo(self):
        self.app.registar_historico_codigo()
        self.app.campo_oculto = False
        if not any(it.get("tipo") == "minicampo" for it in self.app.layout_items):
            mod = getattr(self.app, "modalidade_atual", "futebol")
            nomes_campo = {
                "voleibol": "Campo Voleibol (Zonas 1-6)",
                "basquetebol": "Court Basquetebol",
                "andebol": "Campo Andebol (6m / 9m)",
                "futebol": "Campo 18 Zonas"
            }
            self.app.layout_items.insert(0, {"tipo": "minicampo", "nome": nomes_campo.get(mod, "Campo 18 Zonas"), "x": 12, "y": 10, "w": 256, "h": 134})
        self.reconstruir_widgets()
        self.app.sincronizar_estruturas_desde_canvas()


class TimelineWidget(QWidget):
    def __init__(self, app_ref):
        super().__init__()
        self.app = app_ref
        self.setMinimumHeight(95)
        self.altura_regua = 30
        self.modo_interacao = None
        self.clique_offset = 0.0
        self.setAttribute(Qt.WidgetAttribute.WA_OpaquePaintEvent, True)

    def obter_limites_visiveis(self):
        metade = self.app.janela_zoom_segundos / 2.0
        return self.app.tempo_atual - metade, self.app.tempo_atual + metade

    def tempo_para_xf(self, t, largura):
        t_ini, _ = self.obter_limites_visiveis()
        return ((t - t_ini) / self.app.janela_zoom_segundos) * float(largura)

    def x_para_tempo(self, x, largura):
        t_ini, _ = self.obter_limites_visiveis()
        return t_ini + (float(x) / max(1.0, float(largura))) * self.app.janela_zoom_segundos

    def obter_pistas_ativas_dinamicas(self):
        ativas = []
        for ev in self.app.eventos_registados:
            if ev["acao"] not in ativas:
                ativas.append(ev["acao"])
        if self.app.momento_ativo and self.app.momento_ativo not in ativas:
            ativas.append(self.app.momento_ativo)
        for ac in self.app.acoes_em_gravacao.keys():
            if ac not in ativas:
                ativas.append(ac)
        return ativas

    def paintEvent(self, event):
        p = QPainter(self)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        p.setRenderHint(QPainter.RenderHint.TextAntialiasing, True)
        w, h = self.width(), self.height()

        p.fillRect(0, 0, w, h, QColor("#000000"))
        p.fillRect(0, 0, w, self.altura_regua, QColor("#07070C"))
        p.setPen(QPen(QColor("#222230"), 1.0))
        p.drawLine(0, self.altura_regua, w, self.altura_regua)

        if w <= 20 or self.app.duracao_total <= 0:
            p.setPen(QColor("#71717A"))
            p.setFont(QFont("Helvetica Neue", 11, QFont.Weight.Bold))
            p.drawText(QRect(0, 0, w, h), Qt.AlignmentFlag.AlignCenter, "Carregue um ficheiro de vídeo para ativar a Linha de Tempo")
            return

        t_ini, t_fim = self.obter_limites_visiveis()
        z = self.app.janela_zoom_segundos

        if z <= 25:
            passo_maior, passo_menor = 2, 0.5
        elif z <= 60:
            passo_maior, passo_menor = 5, 1.0
        elif z <= 140:
            passo_maior, passo_menor = 10, 2.0
        else:
            passo_maior, passo_menor = 30, 5.0

        primeiro_menor = math.floor(t_ini / passo_menor) * passo_menor
        t_tick = primeiro_menor
        p.setPen(QPen(QColor("#2E2E40"), 1.0))
        while t_tick <= t_fim:
            if t_tick >= 0:
                xf = self.tempo_para_xf(t_tick, w)
                if 0.0 <= xf <= w:
                    p.drawLine(QPointF(xf, self.altura_regua - 5.0), QPointF(xf, self.altura_regua))
            t_tick += passo_menor

        primeiro_maior = int(t_ini // passo_maior) * passo_maior
        f_timecode = QFont("SF Mono", 10, QFont.Weight.Bold)
        f_timecode.setStyleHint(QFont.StyleHint.Monospace)
        p.setFont(f_timecode)

        for s in range(primeiro_maior, int(t_fim) + passo_maior, passo_maior):
            if s < 0:
                continue
            xf = self.tempo_para_xf(s, w)
            if -35.0 <= xf <= w + 35.0:
                p.setPen(QPen(QColor("#0D0D16"), 1.0))
                p.drawLine(QPointF(xf, self.altura_regua + 1.0), QPointF(xf, h))

                p.setPen(QPen(QColor("#38BDF8"), 1.5))
                p.drawLine(QPointF(xf, self.altura_regua - 9.0), QPointF(xf, self.altura_regua))

                p.setPen(QColor("#FFFFFF"))
                txt_t = self.app.formatar_tempo(s, False)
                p.drawText(QRectF(xf - 30.0, 2.0, 60.0, 17.0), Qt.AlignmentFlag.AlignCenter, txt_t)

        pistas_ativas = self.obter_pistas_ativas_dinamicas()
        if not pistas_ativas:
            p.setPen(QColor("#52525B"))
            p.setFont(QFont("Helvetica Neue", 11))
            p.drawText(
                QRect(0, self.altura_regua, w, h - self.altura_regua),
                Qt.AlignmentFlag.AlignCenter,
                "Linha de Tempo Pronta — Os cortes surgirão aqui automaticamente à medida que forem registados"
            )
        else:
            altura_pista = min(38.0, max(22.0, (h - self.altura_regua - 6) / max(1, len(pistas_ativas))))

            for i in range(len(pistas_ativas)):
                y_sep = self.altura_regua + (i + 1) * altura_pista
                p.setPen(QPen(QColor("#11111A"), 1.0))
                p.drawLine(QPointF(0, y_sep), QPointF(w, y_sep))

            f_clip = QFont("Helvetica Neue", 10, QFont.Weight.Bold)
            p.setFont(f_clip)

            for idx, ev in enumerate(self.app.eventos_registados):
                if ev["fim"] < t_ini or ev["inicio"] > t_fim:
                    continue
                acao = ev["acao"]
                if acao in pistas_ativas:
                    i = pistas_ativas.index(acao)
                    y_topo = self.altura_regua + i * altura_pista + 2.0
                    alt_barra = altura_pista - 4.0

                    x1 = max(0.0, self.tempo_para_xf(ev["inicio"], w))
                    x2 = min(float(w), self.tempo_para_xf(ev["fim"], w))
                    larg_bloco = max(6.0, x2 - x1)

                    cor_hex = self.app.todas_pistas.get(acao, {}).get("cor", "#0284C7")
                    cor_base = QColor(cor_hex)

                    rect_clip = QRectF(x1, y_topo, larg_bloco, alt_barra)
                    grad_clip = QLinearGradient(x1, y_topo, x1, y_topo + alt_barra)
                    grad_clip.setColorAt(0.0, cor_base.lighter(115))
                    grad_clip.setColorAt(1.0, cor_base.darker(115))
                    p.setBrush(QBrush(grad_clip))

                    selecionado = (idx == self.app.bloco_selecionado)
                    p.setPen(QPen(QColor("#FDE047") if selecionado else QColor(255, 255, 255, 110), 2.2 if selecionado else 1.0))
                    p.drawRoundedRect(rect_clip, 5.0, 5.0)

                    if selecionado and larg_bloco > 14:
                        p.fillRect(QRectF(x1 + 1.0, y_topo + 2.0, 4.0, alt_barra - 4.0), QColor("#FDE047"))
                        p.fillRect(QRectF(x2 - 5.0, y_topo + 2.0, 4.0, alt_barra - 4.0), QColor("#FDE047"))

                    if larg_bloco > 40.0:
                        dur_s = round(ev["fim"] - ev["inicio"], 1)
                        t_i_str = self.app.formatar_tempo(ev["inicio"], False)
                        t_f_str = self.app.formatar_tempo(ev["fim"], False)
                        info_txt = f"{acao}  •  {t_i_str} – {t_f_str} ({dur_s}s)"
                        if ev.get("nota"):
                            info_txt += f"  [{ev['nota']}]"

                        p.setPen(QColor("#FFFFFF"))
                        p.drawText(
                            QRectF(x1 + 8.0, y_topo, larg_bloco - 12.0, alt_barra),
                            Qt.AlignmentFlag.AlignVCenter | Qt.AlignmentFlag.AlignLeft,
                            info_txt
                        )

            t_atual = self.app.tempo_atual
            if self.app.momento_ativo in pistas_ativas:
                i = pistas_ativas.index(self.app.momento_ativo)
                y_topo = self.altura_regua + i * altura_pista + 2.0
                alt_barra = altura_pista - 4.0
                x1 = max(0.0, self.tempo_para_xf(self.app.inicio_momento_ativo, w))
                x2 = min(float(w), self.tempo_para_xf(t_atual, w))
                larg_rec = max(6.0, x2 - x1)
                cor_m = QColor(self.app.todas_pistas.get(self.app.momento_ativo, {}).get("cor", "#0284C7"))
                p.setBrush(QBrush(cor_m))
                p.setPen(QPen(QColor("#FDE047"), 2.0))
                p.drawRoundedRect(QRectF(x1, y_topo, larg_rec, alt_barra), 5.0, 5.0)
                p.setPen(QColor("#FFFFFF"))
                dur_rec = max(0.0, t_atual - self.app.inicio_momento_ativo)
                p.drawText(
                    QRectF(x1 + 8.0, y_topo, max(140.0, larg_rec), alt_barra),
                    Qt.AlignmentFlag.AlignVCenter | Qt.AlignmentFlag.AlignLeft,
                    f"REC • {self.app.momento_ativo} ({dur_rec:.1f}s)"
                )

            for ac_nome, d_rec in self.app.acoes_em_gravacao.items():
                if ac_nome in pistas_ativas:
                    i = pistas_ativas.index(ac_nome)
                    t_ini_ac = d_rec["inicio"]
                    y_topo = self.altura_regua + i * altura_pista + 2.0
                    alt_barra = altura_pista - 4.0
                    x1 = max(0.0, self.tempo_para_xf(t_ini_ac, w))
                    x2 = min(float(w), self.tempo_para_xf(t_atual, w))
                    larg_rec = max(6.0, x2 - x1)
                    cor_a = QColor(self.app.todas_pistas.get(ac_nome, {}).get("cor", "#E11D48"))
                    p.setBrush(QBrush(cor_a))
                    p.setPen(QPen(QColor("#FFFFFF"), 2.0, Qt.PenStyle.DashLine))
                    p.drawRoundedRect(QRectF(x1, y_topo, larg_rec, alt_barra), 5.0, 5.0)
                    p.setPen(QColor("#FFFFFF"))
                    dur_rec = max(0.0, t_atual - t_ini_ac)
                    p.drawText(
                        QRectF(x1 + 8.0, y_topo, max(150.0, larg_rec), alt_barra),
                        Qt.AlignmentFlag.AlignVCenter | Qt.AlignmentFlag.AlignLeft,
                        f"CLIQUE ATIVO • {ac_nome} ({dur_rec:.1f}s)"
                    )

        x_play = float(w) / 2.0
        p.setPen(QPen(QColor("#F59E0B"), 2.0))
        p.drawLine(QPointF(x_play, 0), QPointF(x_play, h))

        cabeca_agulha = QPolygonF([
            QPointF(x_play - 6.5, 0.0),
            QPointF(x_play + 6.5, 0.0),
            QPointF(x_play + 6.5, 6.0),
            QPointF(x_play, 12.5),
            QPointF(x_play - 6.5, 6.0)
        ])
        p.setBrush(QBrush(QColor("#F59E0B")))
        p.setPen(QPen(QColor("#FFFFFF"), 1.0))
        p.drawPolygon(cabeca_agulha)

    def mousePressEvent(self, event):
        if self.app.duracao_total <= 0:
            return
        x, y = event.position().x(), event.position().y()
        w, h = self.width(), self.height()

        self.app.playlist_ativa = []
        tempo_clicado = self.x_para_tempo(x, w)

        if y <= self.altura_regua:
            self.app.ir_para_tempo(tempo_clicado)
            return

        pistas_ativas = self.obter_pistas_ativas_dinamicas()
        if not pistas_ativas:
            self.app.ir_para_tempo(tempo_clicado)
            return

        altura_pista = min(38.0, max(22.0, (h - self.altura_regua - 6) / max(1, len(pistas_ativas))))
        pista_clicada = int((y - self.altura_regua) // altura_pista)

        self.app.bloco_selecionado = None
        self.modo_interacao = None

        if 0 <= pista_clicada < len(pistas_ativas):
            acao_alvo = pistas_ativas[pista_clicada]
            for idx, ev in enumerate(self.app.eventos_registados):
                if ev["acao"] == acao_alvo:
                    x1 = self.tempo_para_xf(ev["inicio"], w)
                    x2 = self.tempo_para_xf(ev["fim"], w)
                    if abs(x - x1) <= 9:
                        self.app.bloco_selecionado, self.modo_interacao = idx, "inicio"
                        self.app.ir_para_tempo(ev["inicio"])
                        break
                    elif abs(x - x2) <= 9:
                        self.app.bloco_selecionado, self.modo_interacao = idx, "fim"
                        self.app.ir_para_tempo(ev["fim"])
                        break
                    elif x1 <= x <= x2:
                        self.app.bloco_selecionado, self.modo_interacao = idx, "mover"
                        self.clique_offset = tempo_clicado - ev["inicio"]
                        break

        if self.app.bloco_selecionado is not None:
            nota = self.app.eventos_registados[self.app.bloco_selecionado]["nota"]
            self.app.entry_nota.setText(nota)
            self.update()
        else:
            self.app.ir_para_tempo(tempo_clicado)

    def mouseMoveEvent(self, event):
        if self.app.duracao_total <= 0:
            return
        x = event.position().x()
        w = self.width()
        tempo_cursor = max(0.0, min(self.app.duracao_total, self.x_para_tempo(x, w)))

        if self.app.bloco_selecionado is not None and self.modo_interacao:
            ev = self.app.eventos_registados[self.app.bloco_selecionado]
            rapido = bool(event.modifiers() & Qt.KeyboardModifier.ShiftModifier)
            fator = 1.0 if rapido else 0.25

            if self.modo_interacao == "inicio":
                alvo = ev["inicio"] + (tempo_cursor - ev["inicio"]) * fator
                if alvo < ev["fim"] - 0.5:
                    ev["inicio"] = round(max(0.0, alvo), 2)
                    self.app.ir_para_tempo(ev["inicio"])
            elif self.modo_interacao == "fim":
                alvo = ev["fim"] + (tempo_cursor - ev["fim"]) * fator
                if alvo > ev["inicio"] + 0.5:
                    ev["fim"] = round(min(self.app.duracao_total, alvo), 2)
                    self.app.ir_para_tempo(ev["fim"])
            elif self.modo_interacao == "mover":
                dur = ev["fim"] - ev["inicio"]
                ev["inicio"] = max(0.0, round(tempo_cursor - self.clique_offset, 2))
                ev["fim"] = round(ev["inicio"] + dur, 2)
                self.app.ir_para_tempo(ev["inicio"])
            self.update()
        else:
            self.app.ir_para_tempo(tempo_cursor)

    def mouseReleaseEvent(self, event):
        if self.modo_interacao:
            self.app.db.sincronizar_lista(self.app.eventos_registados)
        self.modo_interacao = None


class VideoExportWorker(QThread):
    progresso = pyqtSignal(int, str)
    concluido = pyqtSignal(str)
    erro = pyqtSignal(str)

    def __init__(self, app_ref, caminho_video, clips, caminho_saida, usar_title_card, usar_minimapa, usar_mask_3d, fps, modo_rapido_ffmpeg=False):
        super().__init__()
        self.app = app_ref
        self.caminho_video = caminho_video
        self.clips = clips
        self.caminho_saida = caminho_saida
        self.usar_title_card = usar_title_card
        self.usar_minimapa = usar_minimapa
        self.usar_mask_3d = usar_mask_3d
        self.fps = fps
        self.modo_rapido_ffmpeg = modo_rapido_ffmpeg
        self.ffmpeg_bin = shutil.which("ffmpeg")

    def run(self):
        try:
            tem_edicoes = any(
                len(c.get("keyframes", [])) > 0 or len(c.get("efeitos", [])) > 0 or c.get("tipo_clip") == "quadro"
                for c in self.clips
            )
            if self.ffmpeg_bin and (self.modo_rapido_ffmpeg or (not tem_edicoes and not self.usar_title_card)):
                self.exportar_via_ffmpeg_stream_copy()
                return
            self.exportar_com_efeitos()
        except Exception as e:
            self.erro.emit(str(e))

    def exportar_via_ffmpeg_stream_copy(self):
        pasta_temp = tempfile.mkdtemp()
        ficheiro_lista = os.path.join(pasta_temp, "concat_list.txt")
        arquivos_partes = []

        clips_video = [c for c in self.clips if c.get("tipo_clip", "video") != "quadro"]
        total = len(clips_video)
        for i, clip in enumerate(clips_video):
            self.progresso.emit(int((i / max(1, total)) * 80), f"Corte rápido FFmpeg ({i+1}/{total})...")
            out_part = os.path.join(pasta_temp, f"part_{i:03d}.mp4")
            dur = max(0.5, clip["fim"] - clip["inicio"])
            cmd = [
                self.ffmpeg_bin, "-y",
                "-ss", str(clip["inicio"]),
                "-i", self.caminho_video,
                "-t", str(dur),
                "-c", "copy",
                "-avoid_negative_ts", "make_zero",
                out_part
            ]
            subprocess.run(cmd, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
            if os.path.exists(out_part):
                arquivos_partes.append(out_part)

        with open(ficheiro_lista, "w", encoding="utf-8") as f:
            for p in arquivos_partes:
                f.write(f"file '{p}'\n")

        self.progresso.emit(90, "A unir segmentos sem perda de qualidade...")
        cmd_concat = [
            self.ffmpeg_bin, "-y", "-f", "concat", "-safe", "0",
            "-i", ficheiro_lista, "-c", "copy", self.caminho_saida
        ]
        subprocess.run(cmd_concat, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        shutil.rmtree(pasta_temp, ignore_errors=True)
        self.progresso.emit(100, "Concluído!")
        self.concluido.emit(self.caminho_saida)

    def exportar_com_efeitos(self):
        cap = cv2.VideoCapture(self.caminho_video)
        w = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH)) or 1280
        h = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT)) or 720

        proc_ffmpeg = None
        out_cv = None
        if self.ffmpeg_bin:
            cmd = [
                self.ffmpeg_bin, "-y",
                "-f", "rawvideo", "-vcodec", "rawvideo",
                "-s", f"{w}x{h}", "-pix_fmt", "bgr24", "-r", str(self.fps),
                "-i", "-",
                "-c:v", "h264_videotoolbox", "-b:v", "8M",
                "-pix_fmt", "yuv420p", self.caminho_saida
            ]
            proc_ffmpeg = subprocess.Popen(cmd, stdin=subprocess.PIPE, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
        else:
            out_cv = cv2.VideoWriter(self.caminho_saida, cv2.VideoWriter_fourcc(*'mp4v'), self.fps, (w, h))

        def gravar_frame(frame_bgr):
            if proc_ffmpeg and proc_ffmpeg.stdin:
                proc_ffmpeg.stdin.write(frame_bgr.tobytes())
            elif out_cv:
                out_cv.write(frame_bgr)

        mat_h = getattr(self.app, "matriz_homografia", None)
        total_clips = len(self.clips)
        for idx, clip in enumerate(self.clips):
            perc = int((idx / max(1, total_clips)) * 100)
            self.progresso.emit(perc, f"A renderizar corte {idx+1} de {total_clips} ({clip['acao']})...")

            if clip.get("tipo_clip") == "quadro":
                quadro_base = KronosProQtApp.gerar_quadro_tatico_base(w, h, clip["acao"], clip.get("nota", ""))
                kfs = clip.get("keyframes", [])
                efeitos_q = kfs[0]["efeitos"] if kfs else clip.get("efeitos", [])
                dur_q = float(kfs[0]["duracao"] if kfs else clip.get("pausa_s", 4.0))
                fx = KronosProQtApp.desenhar_efeitos_broadcast(quadro_base, efeitos_q, False, False, matriz_homografia=mat_h)
                for _ in range(int(dur_q * self.fps)):
                    gravar_frame(fx)
                continue

            if self.usar_title_card:
                card = np.zeros((h, w, 3), dtype=np.uint8)
                cv2.rectangle(card, (80, h // 2 - 90), (88, h // 2 + 90), (248, 189, 56), -1)
                cv2.putText(card, clip["acao"].upper(), (115, h // 2 - 15), cv2.FONT_HERSHEY_SIMPLEX, 1.5, (255, 255, 255), 2, cv2.LINE_AA)
                sub = " | ".join(clip.get("labels", [])) or f"Tempo: {clip['inicio']}s"
                cv2.putText(card, sub, (115, h // 2 + 35), cv2.FONT_HERSHEY_SIMPLEX, 0.9, (180, 185, 195), 2, cv2.LINE_AA)
                if clip.get("nota"):
                    cv2.putText(card, clip["nota"], (115, h // 2 + 80), cv2.FONT_HERSHEY_SIMPLEX, 0.75, (215, 215, 215), 1, cv2.LINE_AA)
                for _ in range(int(1.8 * self.fps)):
                    gravar_frame(card)

            f_ini = int(clip["inicio"] * self.fps)
            f_fim = int(clip["fim"] * self.fps)
            cap.set(cv2.CAP_PROP_POS_FRAMES, f_ini)

            keyframes = sorted(clip.get("keyframes", []), key=lambda k: k["tempo"])
            kf_disparados = set()

            for f_atual in range(f_ini, f_fim):
                ret, frame = cap.read()
                if not ret:
                    break
                t_frame = f_atual / self.fps
                gravar_frame(frame)

                for k_idx, kf in enumerate(keyframes):
                    if k_idx not in kf_disparados and t_frame >= kf["tempo"] and kf.get("efeitos"):
                        kf_disparados.add(k_idx)
                        fx = KronosProQtApp.desenhar_efeitos_broadcast(
                            frame, kf["efeitos"], self.usar_minimapa, self.usar_mask_3d, kf.get("nota", ""), matriz_homografia=mat_h
                        )
                        dur_pausa = float(kf.get("duracao", 3.0))
                        for _ in range(int(dur_pausa * self.fps)):
                            gravar_frame(fx)

        cap.release()
        if proc_ffmpeg and proc_ffmpeg.stdin:
            proc_ffmpeg.stdin.close()
            proc_ffmpeg.wait()
        if out_cv:
            out_cv.release()

        self.progresso.emit(100, "Concluído!")
        self.concluido.emit(self.caminho_saida)


class StudioNLETimelineWidget(QWidget):
    def __init__(self, app_ref):
        super().__init__()
        self.app = app_ref
        self.setFixedHeight(82)
        self.setMouseTracking(True)
        self.setAttribute(Qt.WidgetAttribute.WA_OpaquePaintEvent, True)
        self.hover_x = None
        self.modo_drag = None
        self.kf_alvo_idx = None
        self.x_inicial_drag = 0.0
        self.val_inicial_drag = 0.0
        self.segmentos_layout = []

    def construir_segmentos(self, clip, w):
        kfs = sorted(clip.get("keyframes", []), key=lambda k: k["tempo"])
        dur_video = max(0.5, clip["fim"] - clip["inicio"])
        dur_pausas = sum(float(kf.get("duracao", 3.0)) for kf in kfs)
        dur_total = dur_video + dur_pausas

        margem_px = 14.0
        larg_util = max(20.0, w - 2 * margem_px)
        px_por_seg = larg_util / max(0.1, dur_total)

        segs = []
        cursor_x = margem_px
        t_atual_v = clip["inicio"]

        for k_idx, kf in enumerate(kfs):
            t_kf = max(clip["inicio"], min(clip["fim"], kf["tempo"]))
            if t_kf > t_atual_v:
                larg_v = (t_kf - t_atual_v) * px_por_seg
                segs.append({"tipo": "video", "x1": cursor_x, "x2": cursor_x + larg_v, "t1": t_atual_v, "t2": t_kf})
                cursor_x += larg_v

            dur_kf = float(kf.get("duracao", 3.0))
            larg_kf = max(28.0, dur_kf * px_por_seg)
            segs.append({"tipo": "kf", "x1": cursor_x, "x2": cursor_x + larg_kf, "kf_idx": k_idx, "kf": kf})
            cursor_x += larg_kf
            t_atual_v = t_kf

        if clip["fim"] > t_atual_v or not segs:
            larg_v = max(10.0, (clip["fim"] - t_atual_v) * px_por_seg)
            segs.append({"tipo": "video", "x1": cursor_x, "x2": cursor_x + larg_v, "t1": t_atual_v, "t2": clip["fim"]})

        self.segmentos_layout = segs
        return segs, px_por_seg

    def tempo_para_x(self, t):
        for seg in self.segmentos_layout:
            if seg["tipo"] == "video" and seg["t1"] <= t <= seg["t2"]:
                frac = (t - seg["t1"]) / max(0.01, seg["t2"] - seg["t1"])
                return seg["x1"] + frac * (seg["x2"] - seg["x1"])
        if self.segmentos_layout:
            return self.segmentos_layout[0]["x1"] if t <= self.segmentos_layout[0].get("t1", 0) else self.segmentos_layout[-1]["x2"]
        return 14.0

    def x_para_alvo(self, x):
        for seg in self.segmentos_layout:
            if seg["x1"] <= x <= seg["x2"]:
                if seg["tipo"] == "kf":
                    return "kf", seg["kf_idx"], seg["kf"]["tempo"]
                frac = (x - seg["x1"]) / max(1.0, seg["x2"] - seg["x1"])
                return "video", None, seg["t1"] + frac * (seg["t2"] - seg["t1"])
        return None, None, None

    def paintEvent(self, event):
        p = QPainter(self)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        w, h = float(self.width()), float(self.height())
        p.fillRect(QRectF(0, 0, w, h), QColor("#050509"))

        if self.app.idx_clip_studio is None or not (0 <= self.app.idx_clip_studio < len(self.app.studio_clips)):
            p.setPen(QColor("#52525B"))
            p.drawText(QRectF(0, 0, w, h), Qt.AlignmentFlag.AlignCenter, "Sincronize um corte para ativar a Timeline do Estúdio")
            return

        clip = self.app.studio_clips[self.app.idx_clip_studio]
        segs, _ = self.construir_segmentos(clip, w)
        em_pausa_ativa = (self.app.stack_visor_studio.currentIndex() == 1 and self.app.idx_keyframe_studio is not None)

        for seg in segs:
            x1, x2 = seg["x1"], seg["x2"]
            larg = max(4.0, x2 - x1)
            if seg["tipo"] == "video":
                video_sel = (not em_pausa_ativa and seg["t1"] <= self.app.studio_tempo_atual <= seg["t2"])
                rect_v = QRectF(x1, 18.0, larg, 46.0)
                p.setBrush(QBrush(QColor("#1E293B" if video_sel else "#0F172A")))
                p.setPen(QPen(QColor("#FACC15" if video_sel else "#38BDF8"), 3.2 if video_sel else 1.5))
                p.drawRoundedRect(rect_v, 5.0, 5.0)
                if larg > 48:
                    p.setPen(QColor("#F8FAFC" if video_sel else "#94A3B8"))
                    p.setFont(QFont("SF Mono", 8, QFont.Weight.Bold))
                    dur_s = round(seg["t2"] - seg["t1"], 1)
                    p.drawText(rect_v.adjusted(6, 0, -6, 0), Qt.AlignmentFlag.AlignCenter, f"VÍDEO ({dur_s}s)")
            else:
                k_idx = seg["kf_idx"]
                kf = seg["kf"]
                sel = (k_idx == self.app.idx_keyframe_studio and em_pausa_ativa)
                rect_kf = QRectF(x1, 12.0, larg, 58.0)
                p.setBrush(QBrush(QColor("#F59E0B" if sel else "#92400E")))
                p.setPen(QPen(QColor("#FACC15" if sel else "#FDE047"), 3.8 if sel else 1.2))
                p.drawRoundedRect(rect_kf, 5.0, 5.0)

                p.fillRect(QRectF(x2 - 5.0, 15.0, 4.0, 52.0), QColor("#FFFFFF"))

                p.setPen(QColor("#000000" if sel else "#FFFFFF"))
                p.setFont(QFont("Helvetica Neue", 9, QFont.Weight.Bold))
                p.drawText(rect_kf.adjusted(3, 0, -6, 0), Qt.AlignmentFlag.AlignCenter, f"P{k_idx+1}\n{kf['duracao']:.1f}s")

        x_play = self.tempo_para_x(self.app.studio_tempo_atual)
        p.setPen(QPen(QColor("#FFFFFF"), 2.2))
        p.drawLine(QPointF(x_play, 4.0), QPointF(x_play, h - 4.0))

        if self.hover_x is not None:
            p.setPen(QPen(QColor("#EF4444"), 1.6))
            p.drawLine(QPointF(self.hover_x, 0.0), QPointF(self.hover_x, h))
            tipo_h, k_idx_h, t_skim = self.x_para_alvo(self.hover_x)
            if t_skim is not None:
                lbl_s = f"P{k_idx_h+1}" if tipo_h == "kf" else self.app.formatar_tempo(t_skim, False)
                p.setBrush(QBrush(QColor("#EF4444")))
                p.setPen(Qt.PenStyle.NoPen)
                p.drawRoundedRect(QRectF(self.hover_x - 24, 1, 48, 13), 3, 3)
                p.setPen(QColor("#FFFFFF"))
                p.setFont(QFont("SF Mono", 8, QFont.Weight.Bold))
                p.drawText(QRectF(self.hover_x - 24, 1, 48, 13), Qt.AlignmentFlag.AlignCenter, lbl_s)

    def mousePressEvent(self, event):
        if self.app.idx_clip_studio is None:
            return
        x = event.position().x()

        for seg in self.segmentos_layout:
            if seg["tipo"] == "kf" and seg["x1"] <= x <= seg["x2"]:
                self.kf_alvo_idx = seg["kf_idx"]
                self.x_inicial_drag = x
                if x >= seg["x2"] - 8.0:
                    self.modo_drag = "dur_kf"
                    self.val_inicial_drag = float(seg["kf"].get("duracao", 3.0))
                self.app.abrir_keyframe_no_editor(seg["kf_idx"], auto_retomar=False)
                self.update()
                return

        tipo, _, t_clicado = self.x_para_alvo(x)
        if tipo == "video" and t_clicado is not None:
            self.app.studio_tempo_atual = round(t_clicado, 2)
            self.app.media_player.setPosition(int(t_clicado * 1000))
            if self.app.stack_visor_studio.currentIndex() == 1:
                self.app.stack_visor_studio.setCurrentIndex(0)
                self.app.btn_voltar_video_corte.setVisible(False)
            self.update()

    def mouseMoveEvent(self, event):
        if self.app.idx_clip_studio is None:
            return
        clip = self.app.studio_clips[self.app.idx_clip_studio]
        x = event.position().x()
        self.hover_x = x

        if self.modo_drag == "dur_kf" and self.kf_alvo_idx is not None:
            kf = clip["keyframes"][self.kf_alvo_idx]
            nova_dur = max(0.5, min(20.0, self.val_inicial_drag + (x - self.x_inicial_drag) / 18.0))
            kf["duracao"] = round(nova_dur, 1)
            self.app.spin_duracao_kf.setValue(kf["duracao"])
            self.update()
            return

        if self.app.media_player.playbackState() != QMediaPlayer.PlaybackState.PlayingState:
            tipo, k_idx, t_skim = self.x_para_alvo(x)
            if tipo == "kf" and k_idx is not None:
                self.app.previsualizar_keyframe_temporario(k_idx)
            elif tipo == "video" and t_skim is not None:
                self.app.mostrar_frame_skim_instantaneo(t_skim)
        self.update()

    def mouseReleaseEvent(self, event):
        self.modo_drag = None
        self.kf_alvo_idx = None
        self.app.atualizar_lista_keyframes_ui()

    def leaveEvent(self, event):
        self.hover_x = None
        if self.app.idx_clip_studio is not None and self.app.media_player.playbackState() != QMediaPlayer.PlaybackState.PlayingState:
            em_edicao = (
                self.app.canvas_studio.efeito_rascunho is not None
                or (self.app.idx_keyframe_studio is not None and self.app.btn_voltar_video_corte.isVisible())
            )
            if em_edicao:
                self.app.stack_visor_studio.setCurrentIndex(1)
                self.app.atualizar_preview_estudio()
            else:
                self.app.stack_visor_studio.setCurrentIndex(0)
                self.app.media_player.setPosition(int(self.app.studio_tempo_atual * 1000))
        self.update()
        super().leaveEvent(event)


class EditarTextoTaticoDialog(QDialog):
    def __init__(self, parent, dados_existentes=None):
        super().__init__(parent)
        self.setWindowTitle("Editor de Texto e Caixa Tática — Vision Studio")
        self.resize(440, 480)
        d = dados_existentes or {}

        lay = QVBoxLayout(self)
        form = QFormLayout()

        self.txt_conteudo = QTextEdit(d.get("texto", ""))
        self.txt_conteudo.setPlaceholderText("Escreva o título, descrição ou instrução tática...")
        self.txt_conteudo.setMaximumHeight(95)
        form.addRow("Texto / Descrição:", self.txt_conteudo)

        self.cb_fonte = QComboBox()
        fontes = ["Helvetica Neue", "Arial", "SF Pro Display", "Georgia", "Courier New", "Impact"]
        self.cb_fonte.addItems(fontes)
        if d.get("fonte") in fontes:
            self.cb_fonte.setCurrentText(d["fonte"])
        form.addRow("Tipo de Letra:", self.cb_fonte)

        h_estilo = QHBoxLayout()
        self.sp_tam = QSpinBox()
        self.sp_tam.setRange(10, 96)
        self.sp_tam.setValue(int(d.get("tamanho", 24)))
        h_estilo.addWidget(self.sp_tam)
        self.chk_bold = QCheckBox("Negrito")
        self.chk_bold.setChecked(d.get("negrito", True))
        h_estilo.addWidget(self.chk_bold)
        self.chk_italic = QCheckBox("Itálico")
        self.chk_italic.setChecked(d.get("italico", False))
        h_estilo.addWidget(self.chk_italic)
        form.addRow("Tamanho e Estilo:", h_estilo)

        self.cor_texto = d.get("cor_texto", "#FFFFFF")
        self.btn_cor_txt = QPushButton(self.cor_texto)
        self.btn_cor_txt.setStyleSheet(f"background-color: {self.cor_texto}; color: #000000; border: 1px solid #38BDF8;")
        self.btn_cor_txt.clicked.connect(self.escolher_cor_texto)
        form.addRow("Cor da Letra:", self.btn_cor_txt)

        bg_atual = d.get("cor_fundo", "#0B0F19")
        self.cor_fundo = "#0B0F19" if bg_atual == "transparent" else bg_atual
        self.btn_cor_bg = QPushButton(self.cor_fundo)
        self.btn_cor_bg.setStyleSheet(f"background-color: {self.cor_fundo}; color: #FFFFFF; border: 1px solid #38BDF8;")
        self.btn_cor_bg.clicked.connect(self.escolher_cor_fundo)
        form.addRow("Cor do Fundo:", self.btn_cor_bg)

        self.chk_bg_transp = QCheckBox("Fundo Transparente (Apenas Texto)")
        self.chk_bg_transp.setChecked(bg_atual == "transparent" or d.get("opacidade", 82) == 0)
        form.addRow("", self.chk_bg_transp)

        self.sl_opacidade = QSlider(Qt.Orientation.Horizontal)
        self.sl_opacidade.setRange(0, 100)
        self.sl_opacidade.setValue(int(d.get("opacidade", 82)))
        form.addRow("Opacidade do Fundo (%):", self.sl_opacidade)

        self.sp_largura = QSpinBox()
        self.sp_largura.setRange(10, 95)
        self.sp_largura.setSuffix(" % do ecrã")
        self.sp_largura.setValue(int(d.get("largura_pct", 34)))
        form.addRow("Zona / Largura da Caixa:", self.sp_largura)

        self.cb_align = QComboBox()
        self.cb_align.addItems(["Esquerda", "Centro", "Direita"])
        self.cb_align.setCurrentText(d.get("alinhamento", "Esquerda"))
        form.addRow("Alinhamento:", self.cb_align)

        lay.addLayout(form)

        h_btns = QHBoxLayout()
        b_canc = QPushButton("Cancelar")
        b_canc.clicked.connect(self.reject)
        h_btns.addWidget(b_canc)

        b_ok = QPushButton("Aplicar Caixa de Texto")
        b_ok.setObjectName("SuccessBtn")
        b_ok.clicked.connect(self.accept)
        h_btns.addWidget(b_ok)
        lay.addLayout(h_btns)

    def escolher_cor_texto(self):
        c = QColorDialog.getColor(QColor(self.cor_texto), self, "Cor da Letra")
        if c.isValid():
            self.cor_texto = c.name().upper()
            self.btn_cor_txt.setText(self.cor_texto)
            self.btn_cor_txt.setStyleSheet(f"background-color: {self.cor_texto}; color: #000000; border: 1px solid white;")

    def escolher_cor_fundo(self):
        c = QColorDialog.getColor(QColor(self.cor_fundo), self, "Cor do Fundo da Caixa")
        if c.isValid():
            self.cor_fundo = c.name().upper()
            self.btn_cor_bg.setText(self.cor_fundo)
            self.btn_cor_bg.setStyleSheet(f"background-color: {self.cor_fundo}; color: #FFFFFF; border: 1px solid white;")

    def obter_dados(self):
        return {
            "texto": self.txt_conteudo.toPlainText().strip(),
            "fonte": self.cb_fonte.currentText(),
            "tamanho": self.sp_tam.value(),
            "negrito": self.chk_bold.isChecked(),
            "italico": self.chk_italic.isChecked(),
            "cor_texto": self.cor_texto,
            "cor_fundo": "transparent" if self.chk_bg_transp.isChecked() else self.cor_fundo,
            "opacidade": 0 if self.chk_bg_transp.isChecked() else self.sl_opacidade.value(),
            "largura_pct": self.sp_largura.value(),
            "alinhamento": self.cb_align.currentText()
        }


class StudioCanvasWidget(QLabel):
    def __init__(self, app_ref):
        super().__init__()
        self.app = app_ref
        self.setAlignment(Qt.AlignmentFlag.AlignCenter)
        self.setStyleSheet("background-color: #000000;")
        self.setCursor(Qt.CursorShape.CrossCursor)
        self.setMouseTracking(True)

        self.efeito_selecionado_idx = None
        self.sub_alvo_drag = None
        self.efeito_rascunho = None
        self.offset_drag = (0.0, 0.0)
        self.pos_laser_px = None

    def paintEvent(self, event):
        super().paintEvent(event)
        if self.app.ferramenta_studio == "laser" and self.pos_laser_px is not None:
            p = QPainter(self)
            p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
            p.setBrush(QBrush(QColor(239, 68, 68, 195)))
            p.setPen(QPen(QColor("#FFFFFF"), 2.0))
            p.drawEllipse(self.pos_laser_px, 9.0, 9.0)
            p.end()

    def converter_para_coord_imagem(self, pos_rato):
        pix = self.pixmap()
        if not pix or pix.isNull():
            return pos_rato.x() / max(1, self.width()), pos_rato.y() / max(1, self.height())
        dpr = pix.devicePixelRatio()
        pw, ph = pix.width() / dpr, pix.height() / dpr
        ox = (self.width() - pw) / 2.0
        oy = (self.height() - ph) / 2.0
        nx = max(0.0, min(1.0, (pos_rato.x() - ox) / max(1.0, pw)))
        ny = max(0.0, min(1.0, (pos_rato.y() - oy) / max(1.0, ph)))
        return nx, ny

    def procurar_hit_em_objeto(self, ef, nx, ny):
        if ef.get("tipo") == "texto" and "_rect_norm" in ef:
            rx, ry, rw, rh = ef["_rect_norm"]
            if (rx - 0.015) <= nx <= (rx + rw + 0.015) and (ry - 0.015) <= ny <= (ry + rh + 0.015):
                if abs(nx - rx) <= 0.022 or abs(nx - (rx + rw)) <= 0.022:
                    return "borda_horiz"
                if abs(ny - ry) <= 0.022 or abs(ny - (ry + rh)) <= 0.022:
                    return "borda_vert"
                return "centro"
            return None
        raio_hit = 0.042
        if "nx" in ef and "ny" in ef:
            if math.hypot(nx - ef["nx"], ny - ef["ny"]) <= raio_hit:
                return "centro"
        elif "nx1" in ef and "ny1" in ef and "nx2" in ef and "ny2" in ef:
            if math.hypot(nx - ef["nx1"], ny - ef["ny1"]) <= raio_hit:
                return "p1"
            if math.hypot(nx - ef["nx2"], ny - ef["ny2"]) <= raio_hit:
                return "p2"
            mx, my = (ef["nx1"] + ef["nx2"]) / 2.0, (ef["ny1"] + ef["ny2"]) / 2.0
            if math.hypot(nx - mx, ny - my) <= raio_hit * 1.3:
                return "ambos"
        return None

    def procurar_efeito_no_ponto(self, efeitos, nx, ny):
        for idx in range(len(efeitos) - 1, -1, -1):
            parte = self.procurar_hit_em_objeto(efeitos[idx], nx, ny)
            if parte is not None:
                return idx, parte
        return None, None

    def mousePressEvent(self, event):
        kf = self.app.obter_keyframe_studio_ativo()
        if kf is None or self.app.frame_studio_original is None:
            return
        nx, ny = self.converter_para_coord_imagem(event.position())

        if getattr(self.app, "modo_calibracao", False) and event.button() == Qt.MouseButton.LeftButton:
            self.app.pontos_calibracao.append((nx, ny))
            if len(self.app.pontos_calibracao) >= 4:
                self.app.finalizar_calibracao_homografia()
            else:
                self.app.atualizar_preview_estudio()
            return

        efeitos = kf.setdefault("efeitos", [])
        ferr = self.app.ferramenta_studio

        if event.button() == Qt.MouseButton.RightButton:
            if self.efeito_rascunho is not None:
                self.app.descartar_efeito_rascunho()
                return
            idx_hit, _ = self.procurar_efeito_no_ponto(efeitos, nx, ny)
            if idx_hit is not None:
                efeitos.pop(idx_hit)
                self.efeito_selecionado_idx = None
                self.app.atualizar_preview_estudio()
            return

        if ferr == "laser":
            self.pos_laser_px = event.position()
            self.update()
            return

        if self.efeito_rascunho is not None:
            parte_r = self.procurar_hit_em_objeto(self.efeito_rascunho, nx, ny)
            if parte_r is not None:
                self.sub_alvo_drag = f"rascunho_{parte_r}"
                if parte_r == "centro":
                    self.offset_drag = (nx - self.efeito_rascunho["nx"], ny - self.efeito_rascunho["ny"])
                elif parte_r == "ambos":
                    self.offset_drag = (nx, ny)
                return

        idx_hit, parte = self.procurar_efeito_no_ponto(efeitos, nx, ny)
        if idx_hit is not None and (ferr != "linha" or efeitos[idx_hit]["tipo"] == "linha"):
            self.efeito_selecionado_idx = idx_hit
            self.sub_alvo_drag = parte
            ef = efeitos[idx_hit]
            if parte == "centro":
                self.offset_drag = (nx - ef["nx"], ny - ef["ny"])
            elif parte == "ambos":
                self.offset_drag = (nx, ny)
            self.app.atualizar_preview_estudio(idx_selecionado=self.efeito_selecionado_idx)
            return

        if ferr == "texto":
            dlg = EditarTextoTaticoDialog(self)
            if dlg.exec():
                dados_t = dlg.obter_dados()
                if dados_t["texto"]:
                    dados_t.update({"tipo": "texto", "nx": nx, "ny": ny})
                    self.efeito_rascunho = dados_t
                    self.app.barra_teste_efeito.setVisible(True)
                    self.app.atualizar_preview_estudio(efeito_preview=self.efeito_rascunho)
            return

        cor_padrao = self.app.cores_efeitos.get(ferr, "#00E5FF")
        if ferr == "linha":
            if self.efeito_rascunho is not None and self.efeito_rascunho.get("tipo") == "linha":
                self.efeito_rascunho["pontos"].append((nx, ny))
            else:
                self.efeito_rascunho = {"tipo": "linha", "pontos": [(nx, ny)], "cor": cor_padrao, "opacidade": 36}
            self.app.barra_teste_efeito.setVisible(True)
            self.app.atualizar_preview_estudio(efeito_preview=self.efeito_rascunho)
            return

        self.sub_alvo_drag = "criar_novo"
        if ferr in ("seta", "cone", "sombra", "ghost", "zoom"):
            self.efeito_rascunho = {
                "tipo": ferr, "cor": cor_padrao,
                "nx1": nx, "ny1": ny,
                "nx2": min(0.92, nx + 0.14), "ny2": max(0.08, ny - 0.10)
            }
        elif ferr == "livre":
            self.efeito_rascunho = {"tipo": "livre", "cor": cor_padrao, "pontos": [(nx, ny)]}
        else:
            self.efeito_rascunho = {"tipo": ferr, "cor": cor_padrao, "nx": nx, "ny": ny}

        self.app.barra_teste_efeito.setVisible(True)
        self.app.atualizar_preview_estudio(efeito_preview=self.efeito_rascunho)

    def mouseDoubleClickEvent(self, event):
        kf = self.app.obter_keyframe_studio_ativo()
        if kf is None:
            return
        nx, ny = self.converter_para_coord_imagem(event.position())
        if self.efeito_rascunho is not None and self.efeito_rascunho.get("tipo") == "texto":
            if self.procurar_hit_em_objeto(self.efeito_rascunho, nx, ny) is not None:
                dlg = EditarTextoTaticoDialog(self, self.efeito_rascunho)
                if dlg.exec():
                    self.efeito_rascunho.update(dlg.obter_dados())
                    self.app.atualizar_preview_estudio(efeito_preview=self.efeito_rascunho)
                return
        idx_hit, _ = self.procurar_efeito_no_ponto(kf.get("efeitos", []), nx, ny)
        if idx_hit is not None and kf["efeitos"][idx_hit].get("tipo") == "texto":
            dlg = EditarTextoTaticoDialog(self, kf["efeitos"][idx_hit])
            if dlg.exec():
                kf["efeitos"][idx_hit].update(dlg.obter_dados())
                self.app.atualizar_preview_estudio(idx_selecionado=idx_hit)

    def mouseMoveEvent(self, event):
        kf = self.app.obter_keyframe_studio_ativo()
        if kf is None or self.app.frame_studio_original is None:
            return
        ferr = self.app.ferramenta_studio
        if ferr == "laser":
            self.pos_laser_px = event.position()
            self.update()
            return

        nx, ny = self.converter_para_coord_imagem(event.position())
        efeitos = kf.get("efeitos", [])

        if not (event.buttons() & Qt.MouseButton.LeftButton):
            hit_r = self.procurar_hit_em_objeto(self.efeito_rascunho, nx, ny) if self.efeito_rascunho else None
            idx_hit, parte_h = self.procurar_efeito_no_ponto(efeitos, nx, ny)
            alvo = hit_r or parte_h
            if alvo == "borda_horiz":
                self.setCursor(Qt.CursorShape.SizeHorCursor)
            elif alvo == "borda_vert":
                self.setCursor(Qt.CursorShape.SizeVerCursor)
            elif alvo is not None:
                self.setCursor(Qt.CursorShape.OpenHandCursor)
            else:
                self.setCursor(Qt.CursorShape.CrossCursor)
            return

        if self.sub_alvo_drag == "criar_novo" and self.efeito_rascunho is not None:
            tp = self.efeito_rascunho["tipo"]
            if tp in ("seta", "cone", "sombra", "ghost", "zoom"):
                self.efeito_rascunho["nx2"], self.efeito_rascunho["ny2"] = nx, ny
            elif tp == "livre":
                self.efeito_rascunho["pontos"].append((nx, ny))
            else:
                self.efeito_rascunho["nx"], self.efeito_rascunho["ny"] = nx, ny
            self.app.atualizar_preview_estudio(efeito_preview=self.efeito_rascunho)
            return

        if self.sub_alvo_drag and self.sub_alvo_drag.startswith("rascunho_") and self.efeito_rascunho is not None:
            parte = self.sub_alvo_drag.replace("rascunho_", "")
            self._aplicar_drag_objeto(self.efeito_rascunho, parte, nx, ny)
            self.app.atualizar_preview_estudio(efeito_preview=self.efeito_rascunho)
            return

        if self.efeito_selecionado_idx is not None and 0 <= self.efeito_selecionado_idx < len(efeitos):
            self._aplicar_drag_objeto(efeitos[self.efeito_selecionado_idx], self.sub_alvo_drag, nx, ny)
            self.app.atualizar_preview_estudio(idx_selecionado=self.efeito_selecionado_idx)

    def _aplicar_drag_objeto(self, ef, parte, nx, ny):
        self.setCursor(Qt.CursorShape.ClosedHandCursor)
        if parte == "centro":
            ef["nx"] = max(0.0, min(1.0, nx - self.offset_drag[0]))
            ef["ny"] = max(0.0, min(1.0, ny - self.offset_drag[1]))
        elif parte == "borda_horiz":
            ef["largura_pct"] = max(12, min(92, int(abs(nx - ef["nx"]) * 200)))
        elif parte == "borda_vert":
            ef["altura_pct"] = max(6, min(80, int(abs(ny - ef["ny"]) * 200)))
        elif parte == "p1":
            ef["nx1"], ef["ny1"] = nx, ny
        elif parte == "p2":
            ef["nx2"], ef["ny2"] = nx, ny
        elif parte == "ambos":
            dx, dy = nx - self.offset_drag[0], ny - self.offset_drag[1]
            ef["nx1"] = max(0.0, min(1.0, ef["nx1"] + dx))
            ef["ny1"] = max(0.0, min(1.0, ef["ny1"] + dy))
            ef["nx2"] = max(0.0, min(1.0, ef["nx2"] + dx))
            ef["ny2"] = max(0.0, min(1.0, ef["ny2"] + dy))
            self.offset_drag = (nx, ny)

    def mouseReleaseEvent(self, event):
        self.sub_alvo_drag = None
        self.setCursor(Qt.CursorShape.CrossCursor)


class KronosProQtApp(QMainWindow):
    def __init__(self, lic_mgr=None):
        super().__init__()
        self.lic_mgr = lic_mgr or LicenseManager()
        self.setWindowTitle(f"{APP_NAME} v{APP_VERSION} — {APP_SUBTITLE}")
        self.setMinimumSize(1024, 640)
        ecra = QApplication.primaryScreen()
        if ecra is not None:
            self.setGeometry(ecra.availableGeometry())
        else:
            self.resize(1366, 800)
        self.setStyleSheet(STYLESHEET)

        self.db = TacticalDatabase()

        self.caminho_video = ""
        self.fps = 25.0
        self.duracao_total = 0.0
        self.tempo_atual = 0.0

        self.lista_velocidades = [0.25, 0.5, 1.0, 1.5, 2.0, 4.0, 8.0]
        self.idx_velocidade = 2
        self.janela_zoom_segundos = 45.0
        self.arrastando_slider = False

        self.acoes_em_gravacao = {}

        self.layout_items = [
            {"tipo": "minicampo", "nome": "Campo 18 Zonas", "x": 12, "y": 10, "w": 256, "h": 134},
            {"tipo": "momento", "nome": "Organização Ofensiva", "atalho": "1", "cor": "#0284C7", "x": 12, "y": 154, "w": 256, "h": 36, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
            {"tipo": "momento", "nome": "Transição Defensiva", "atalho": "2", "cor": "#D97706", "x": 12, "y": 194, "w": 256, "h": 36, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
            {"tipo": "momento", "nome": "Organização Defensiva", "atalho": "3", "cor": "#334155", "x": 12, "y": 234, "w": 256, "h": 36, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
            {"tipo": "momento", "nome": "Transição Ofensiva", "atalho": "4", "cor": "#0D9488", "x": 12, "y": 274, "w": 256, "h": 36, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
            {"tipo": "momento", "nome": "Bola Parada", "atalho": "5", "cor": "#7C3AED", "x": 12, "y": 314, "w": 256, "h": 36, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
            {"tipo": "acao", "nome": "Remate", "atalho": "a", "cor": "#E11D48", "x": 12, "y": 362, "w": 124, "h": 44, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 10.0, "pos_tipo": "segundos", "pos_val": 10.0},
            {"tipo": "acao", "nome": "Passe Chave", "atalho": "p", "cor": "#059669", "x": 144, "y": 362, "w": 124, "h": 44, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 10.0, "pos_tipo": "segundos", "pos_val": 10.0},
            {"tipo": "acao", "nome": "Perda de Posse", "atalho": "f", "cor": "#EA580C", "x": 12, "y": 412, "w": 124, "h": 44, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 10.0, "pos_tipo": "segundos", "pos_val": 10.0, "link_momento": "Transição Defensiva"},
            {"tipo": "acao", "nome": "Recuperação", "atalho": "r", "cor": "#16A34A", "x": 144, "y": 412, "w": 124, "h": 44, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 10.0, "pos_tipo": "segundos", "pos_val": 10.0, "link_momento": "Transição Ofensiva"},
            {"tipo": "acao", "nome": "Cruzamento", "atalho": "c", "cor": "#2563EB", "x": 12, "y": 462, "w": 124, "h": 44, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 10.0, "pos_tipo": "segundos", "pos_val": 10.0},
            {"tipo": "acao", "nome": "Falta", "atalho": "y", "cor": "#CA8A04", "x": 144, "y": 462, "w": 124, "h": 44, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 10.0, "pos_tipo": "segundos", "pos_val": 10.0},
            {"tipo": "label", "nome": "Corredor Esq", "atalho": "", "cor": "#14141D", "x": 12, "y": 518, "w": 82, "h": 28, "fonte": 10, "raio": 4},
            {"tipo": "label", "nome": "Corredor Central", "atalho": "", "cor": "#14141D", "x": 99, "y": 518, "w": 82, "h": 28, "fonte": 10, "raio": 4},
            {"tipo": "label", "nome": "Corredor Dir", "atalho": "", "cor": "#14141D", "x": 186, "y": 518, "w": 82, "h": 28, "fonte": 10, "raio": 4},
            {"tipo": "label", "nome": "Positivo", "atalho": "", "cor": "#14141D", "x": 12, "y": 552, "w": 82, "h": 28, "fonte": 10, "raio": 4},
            {"tipo": "label", "nome": "Negativo", "atalho": "", "cor": "#14141D", "x": 99, "y": 552, "w": 82, "h": 28, "fonte": 10, "raio": 4},
            {"tipo": "label", "nome": "Golo", "atalho": "", "cor": "#14141D", "x": 186, "y": 552, "w": 82, "h": 28, "fonte": 10, "raio": 4}
        ]

        self.historico_codigo_undo = []
        self.historico_codigo_redo = []
        self.campo_oculto = False

        self.modalidade_atual = "futebol"
        self.presets_modalidades = {
            "futebol": copy.deepcopy(self.layout_items),
            "voleibol": [
                {"tipo": "minicampo", "nome": "Campo Voleibol (Zonas 1-9)", "x": 12, "y": 10, "w": 256, "h": 134},
                {"tipo": "momento", "nome": "Rali em Curso", "atalho": "1", "cor": "#0284C7", "x": 12, "y": 154, "w": 256, "h": 34, "fonte": 11, "raio": 5, "pre_tipo": "segundos", "pre_val": 1.0, "pos_tipo": "clique", "pos_val": 0.0},
                {"tipo": "acao", "nome": "Serviço", "atalho": "s", "cor": "#0284C7", "x": 12, "y": 196, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "acao", "nome": "Receção", "atalho": "r", "cor": "#059669", "x": 144, "y": 196, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "acao", "nome": "Ataque", "atalho": "a", "cor": "#E11D48", "x": 12, "y": 244, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "acao", "nome": "Bloco", "atalho": "b", "cor": "#7C3AED", "x": 144, "y": 244, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "acao", "nome": "Defesa", "atalho": "d", "cor": "#D97706", "x": 12, "y": 292, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "acao", "nome": "Ponto / Erro", "atalho": "p", "cor": "#16A34A", "x": 144, "y": 292, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                # Botões de Zona 1 a 9 (alinhados em 3x3)
                {"tipo": "label", "nome": "Zona 5", "atalho": "", "cor": "#14141D", "x": 12, "y": 344, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Zona 7", "atalho": "", "cor": "#14141D", "x": 99, "y": 344, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Zona 4", "atalho": "", "cor": "#14141D", "x": 186, "y": 344, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Zona 6", "atalho": "", "cor": "#14141D", "x": 12, "y": 374, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Zona 8", "atalho": "", "cor": "#14141D", "x": 99, "y": 374, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Zona 3", "atalho": "", "cor": "#14141D", "x": 186, "y": 374, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Zona 1", "atalho": "", "cor": "#14141D", "x": 12, "y": 404, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Zona 9", "atalho": "", "cor": "#14141D", "x": 99, "y": 404, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Zona 2", "atalho": "", "cor": "#14141D", "x": 186, "y": 404, "w": 82, "h": 26, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Positivo", "atalho": "", "cor": "#14141D", "x": 12, "y": 438, "w": 124, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Erro / Negativo", "atalho": "", "cor": "#14141D", "x": 144, "y": 438, "w": 124, "h": 28, "fonte": 10, "raio": 4}
            ],
            "basquetebol": [
                {"tipo": "minicampo", "nome": "Court Basquetebol", "x": 12, "y": 10, "w": 256, "h": 134},
                {"tipo": "momento", "nome": "Ataque Organizado", "atalho": "1", "cor": "#0284C7", "x": 12, "y": 154, "w": 256, "h": 34, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
                {"tipo": "momento", "nome": "Defesa / Transição", "atalho": "2", "cor": "#D97706", "x": 12, "y": 192, "w": 256, "h": 34, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
                {"tipo": "acao", "nome": "Lançamento 2P", "atalho": "q", "cor": "#059669", "x": 12, "y": 236, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 3.0, "pos_tipo": "segundos", "pos_val": 2.0},
                {"tipo": "acao", "nome": "Triplo (3P)", "atalho": "t", "cor": "#E11D48", "x": 144, "y": 236, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 3.0, "pos_tipo": "segundos", "pos_val": 2.0},
                {"tipo": "acao", "nome": "Perda Bola (TO)", "atalho": "o", "cor": "#EA580C", "x": 12, "y": 284, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 3.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "acao", "nome": "Ressalto", "atalho": "r", "cor": "#7C3AED", "x": 144, "y": 284, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 2.0},
                {"tipo": "acao", "nome": "Bloqueio / Roubo", "atalho": "b", "cor": "#2563EB", "x": 12, "y": 332, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 3.0, "pos_tipo": "segundos", "pos_val": 2.0},
                {"tipo": "acao", "nome": "Falta / LL", "atalho": "f", "cor": "#CA8A04", "x": 144, "y": 332, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "label", "nome": "Garrafão", "atalho": "", "cor": "#14141D", "x": 12, "y": 384, "w": 82, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Ala / 3P", "atalho": "", "cor": "#14141D", "x": 99, "y": 384, "w": 82, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Topo / Chave", "atalho": "", "cor": "#14141D", "x": 186, "y": 384, "w": 82, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Convertido", "atalho": "", "cor": "#14141D", "x": 12, "y": 418, "w": 124, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Falhado", "atalho": "", "cor": "#14141D", "x": 144, "y": 418, "w": 124, "h": 28, "fonte": 10, "raio": 4}
            ],
            "andebol": [
                {"tipo": "minicampo", "nome": "Campo Andebol (6m / 9m)", "x": 12, "y": 10, "w": 256, "h": 134},
                {"tipo": "momento", "nome": "Ataque Posicional", "atalho": "1", "cor": "#0284C7", "x": 12, "y": 154, "w": 256, "h": 34, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
                {"tipo": "momento", "nome": "Defesa / Recuo", "atalho": "2", "cor": "#D97706", "x": 12, "y": 192, "w": 256, "h": 34, "fonte": 11, "raio": 5, "pre_tipo": "clique", "pre_val": 0.0, "pos_tipo": "clique", "pos_val": 0.0},
                {"tipo": "acao", "nome": "Remate 6m / Ponta", "atalho": "a", "cor": "#E11D48", "x": 12, "y": 236, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 3.0, "pos_tipo": "segundos", "pos_val": 2.0},
                {"tipo": "acao", "nome": "Remate 9m / Susp.", "atalho": "w", "cor": "#7C3AED", "x": 144, "y": 236, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 3.0, "pos_tipo": "segundos", "pos_val": 2.0},
                {"tipo": "acao", "nome": "Contra-Ataque", "atalho": "c", "cor": "#059669", "x": 12, "y": 284, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "acao", "nome": "Perda / Falta Téc.", "atalho": "t", "cor": "#EA580C", "x": 144, "y": 284, "w": 124, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 2.0},
                {"tipo": "acao", "nome": "Livre 7m / Exclusão", "atalho": "e", "cor": "#CA8A04", "x": 12, "y": 332, "w": 256, "h": 42, "fonte": 11, "raio": 6, "pre_tipo": "segundos", "pre_val": 2.0, "pos_tipo": "segundos", "pos_val": 3.0},
                {"tipo": "label", "nome": "Ponta Esq", "atalho": "", "cor": "#14141D", "x": 12, "y": 384, "w": 124, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Ponta Dir", "atalho": "", "cor": "#14141D", "x": 144, "y": 384, "w": 124, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Lateral Esq", "atalho": "", "cor": "#14141D", "x": 12, "y": 418, "w": 82, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Central / Pivot", "atalho": "", "cor": "#14141D", "x": 99, "y": 418, "w": 82, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Lateral Dir", "atalho": "", "cor": "#14141D", "x": 186, "y": 418, "w": 82, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Golo", "atalho": "", "cor": "#14141D", "x": 12, "y": 452, "w": 124, "h": 28, "fonte": 10, "raio": 4},
                {"tipo": "label", "nome": "Defesa GR / Fora", "atalho": "", "cor": "#14141D", "x": 144, "y": 452, "w": 124, "h": 28, "fonte": 10, "raio": 4}
            ]
        }

        self.momentos_jogo = {}
        self.categorias_eventos = {}
        self.todas_pistas = {}
        self.lista_labels = []
        self.momento_ativo = None
        self.inicio_momento_ativo = 0.0
        self.labels_ativas = set()
        self.coord_campo_atual = (0.5, 0.5)
        self.zona_campo_atual = "Zona 8 (Corredor Central)"

        self.eventos_registados = []
        self.bloco_selecionado = None
        self.playlist_ativa = []
        self.idx_playlist = 0

        self.studio_clips = []
        self.idx_clip_studio = None
        self.idx_keyframe_studio = None
        self.ferramenta_studio = "anel"
        self.masc_relva_cache = None
        self.matriz_homografia = None
        self.modo_calibracao = False
        self.pontos_calibracao = []
        self.cores_efeitos = {
            "anel": "#00E5FF", "spotlight": "#FFD600", "linha": "#00E5FF",
            "seta": "#FFD600", "cone": "#38BDF8", "sombra": "#F43F5E",
            "ghost": "#4ADE80", "zoom": "#00E5FF", "livre": "#F59E0B", "texto": "#FFFFFF"
        }

        self.frame_studio_original = None
        self.worker_export = None
        self.worker_skim = None
        self.worker_opt = None
        self.studio_tempo_atual = 0.0
        self.studio_keyframes_ja_pausados = set()

        self.media_player = QMediaPlayer(self)
        self.audio_output = QAudioOutput(self)
        self.media_player.setAudioOutput(self.audio_output)
        self.audio_output.setVolume(0.8)
        self.media_player.positionChanged.connect(self.ao_mudar_posicao_gpu)
        self.media_player.durationChanged.connect(self.ao_mudar_duracao_gpu)

        self.timer_pausa_studio = QTimer(self)
        self.timer_pausa_studio.setSingleShot(True)
        self.timer_pausa_studio.timeout.connect(self.retomar_video_studio_apos_pausa)

        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        main_layout = QVBoxLayout(central_widget)
        main_layout.setContentsMargins(0, 0, 0, 0)
        main_layout.setSpacing(0)

        self.construir_barra_superior(main_layout)

        self.tabs = QTabWidget()
        self.tabs.currentChanged.connect(self.ao_alternar_separador_principal)
        main_layout.addWidget(self.tabs, 1)

        self.tab_codificacao = QWidget()
        self.tab_estudio = QWidget()
        self.tab_analytics = QWidget()

        self.tabs.addTab(self.tab_codificacao, "MATCH DESK")
        self.tabs.addTab(self.tab_estudio, "VISION STUDIO")
        self.tabs.addTab(self.tab_analytics, "SPATIAL LAB")

        self.sincronizar_dicionarios_internos()
        self.configurar_tab_codificacao()
        self.configurar_tab_estudio()
        self.configurar_tab_analytics()

        self.media_player.setVideoOutput(self.video_widget)
        QApplication.instance().installEventFilter(self)
        self.projeto_atual_path = ""
        self.alteracoes_por_guardar = False
        QTimer.singleShot(150, self.verificar_recuperacao)

    def ao_alternar_separador_principal(self, idx):
        if not hasattr(self, "btn_play") or not hasattr(self, "video_widget"):
            return
        self.timer_pausa_studio.stop()
        self.media_player.pause()
        self.btn_play.definir_modo("play")
        if hasattr(self, "btn_play_studio"):
            self.btn_play_studio.definir_modo("play")

        if idx == 0:
            self.media_player.setVideoOutput(self.video_widget)
            self.media_player.setPlaybackRate(self.lista_velocidades[self.idx_velocidade])
            self.media_player.setPosition(int(self.tempo_atual * 1000))
        elif idx == 1 and hasattr(self, "video_widget_studio"):
            self.media_player.setVideoOutput(self.video_widget_studio)
            self.media_player.setPlaybackRate(1.0)
            if self.idx_clip_studio is not None and 0 <= self.idx_clip_studio < len(self.studio_clips):
                clip = self.studio_clips[self.idx_clip_studio]
                t_alvo = self.studio_tempo_atual if (clip["inicio"] <= self.studio_tempo_atual <= clip["fim"]) else clip["inicio"]
                self.media_player.setPosition(int(t_alvo * 1000))
        elif idx == 2:
            self.desenhar_heatmap_espacial()

    def sincronizar_dicionarios_internos(self):
        if not hasattr(self, "campo_oculto"):
            self.campo_oculto = False

        self.momentos_jogo.clear()
        self.categorias_eventos.clear()
        self.todas_pistas.clear()
        self.lista_labels.clear()

        for it in self.layout_items:
            if it["tipo"] == "momento":
                self.momentos_jogo[it["nome"]] = it
                self.todas_pistas[it["nome"]] = it
            elif it["tipo"] == "acao":
                self.categorias_eventos[it["nome"]] = it
                self.todas_pistas[it["nome"]] = it
            elif it["tipo"] == "label":
                self.lista_labels.append(it["nome"])

    def _atualizar_ui_apos_edicao_canvas(self):
        if hasattr(self, "combo_heatmap"):
            self.combo_heatmap.blockSignals(True)
            self.combo_heatmap.clear()
            self.combo_heatmap.addItems(["Todas as Ações"] + list(self.todas_pistas.keys()))
            self.combo_heatmap.blockSignals(False)
        if hasattr(self, "combo_etiqueta_spatial"):
            self.combo_etiqueta_spatial.blockSignals(True)
            self.combo_etiqueta_spatial.clear()
            self.combo_etiqueta_spatial.addItems(["Todas as Etiquetas"] + list(self.lista_labels))
            self.combo_etiqueta_spatial.blockSignals(False)
        if hasattr(self, "layout_grupos_dinamico") and hasattr(self, "dash_labels"):
            self.reconstruir_conteudo_painel_direito()
        if hasattr(self, "dash_labels"):
            self.atualizar_dashboard()
        if hasattr(self, "timeline_widget"):
            self.timeline_widget.update()

    def sincronizar_estruturas_desde_canvas(self):
        self.sincronizar_dicionarios_internos()
        QTimer.singleShot(0, self._atualizar_ui_apos_edicao_canvas)

    def descartar_campo_canvas(self):
        if hasattr(self, "freeform_canvas") and self.freeform_canvas is not None:
            self.freeform_canvas.descartar_campo()

    def repor_campo_canvas(self):
        if hasattr(self, "freeform_canvas") and self.freeform_canvas is not None:
            self.freeform_canvas.repor_campo()

    def _aplicar_estado_historico_codigo(self):
        if getattr(self, "freeform_canvas", None) is not None:
            self.freeform_canvas.widget_selecionado = None
            for w in getattr(self, "freeform_canvas", []).widgets_itens:
                w.modo_arrasto = None
            self.freeform_canvas.reconstruir_widgets()
        self.sincronizar_estruturas_desde_canvas()

    def registar_historico_codigo(self):
        if not hasattr(self, "historico_codigo_undo"):
            self.historico_codigo_undo = []
        if not hasattr(self, "historico_codigo_redo"):
            self.historico_codigo_redo = []
        self.historico_codigo_undo.append(copy.deepcopy(self.layout_items))
        if len(self.historico_codigo_undo) > 50:
            self.historico_codigo_undo.pop(0)
        self.historico_codigo_redo.clear()

    def desfazer_janela_codigo(self):
        if not getattr(self, "historico_codigo_undo", None):
            return
        if not hasattr(self, "historico_codigo_redo"):
            self.historico_codigo_redo = []
        self.historico_codigo_redo.append(copy.deepcopy(self.layout_items))
        self.layout_items = self.historico_codigo_undo.pop()
        self.campo_oculto = not any(it.get("tipo") == "minicampo" for it in self.layout_items)
        QTimer.singleShot(0, self._aplicar_estado_historico_codigo)

    def refazer_janela_codigo(self):
        if not getattr(self, "historico_codigo_redo", None):
            return
        if not hasattr(self, "historico_codigo_undo"):
            self.historico_codigo_undo = []
        self.historico_codigo_undo.append(copy.deepcopy(self.layout_items))
        self.layout_items = self.historico_codigo_redo.pop()
        self.campo_oculto = not any(it.get("tipo") == "minicampo" for it in self.layout_items)
        QTimer.singleShot(0, self._aplicar_estado_historico_codigo)

    def eventFilter(self, obj, event):
        if event.type() == QEvent.Type.Resize and obj is getattr(self, "painel_direito", None):
            self.atualizar_escala_grupos_contadores()

        if event.type() == QEvent.Type.KeyPress:
            foco = QApplication.focusWidget()
            if isinstance(foco, (QLineEdit, QTextEdit, QSpinBox, QDoubleSpinBox)):
                return super().eventFilter(obj, event)

            key = event.key()
            mods = event.modifiers()

            if getattr(self, "freeform_canvas", None) and self.freeform_canvas.modo_edicao and self.freeform_canvas.widget_selecionado is not None:
                passo = 10 if (mods & Qt.KeyboardModifier.ShiftModifier) else 1
                if key == Qt.Key.Key_Left:
                    self.freeform_canvas.mover_selecionado_teclado(-passo, 0)
                    return True
                elif key == Qt.Key.Key_Right:
                    self.freeform_canvas.mover_selecionado_teclado(passo, 0)
                    return True
                elif key == Qt.Key.Key_Up:
                    self.freeform_canvas.mover_selecionado_teclado(0, -passo)
                    return True
                elif key == Qt.Key.Key_Down:
                    self.freeform_canvas.mover_selecionado_teclado(0, passo)
                    return True
                elif key == Qt.Key.Key_Backspace:
                    self.freeform_canvas.apagar_selecionado_teclado()
                    return True

            if self.tabs.currentIndex() == 0:
                if key == Qt.Key.Key_Left:
                    passo = -3.0 if (mods & Qt.KeyboardModifier.ShiftModifier) else -15.0
                    self.saltar_segundos(passo)
                    return True
                elif key == Qt.Key.Key_Right:
                    passo = 3.0 if (mods & Qt.KeyboardModifier.ShiftModifier) else 15.0
                    self.saltar_segundos(passo)
                    return True
                elif key == Qt.Key.Key_Space or key == Qt.Key.Key_K:
                    self.alternar_play_pause()
                    return True
                elif key == Qt.Key.Key_L:
                    self.atalho_avancar_l()
                    return True
                elif key == Qt.Key.Key_J:
                    self.saltar_segundos(-15.0)
                    return True

                texto_tecla = event.text().lower()
                if texto_tecla and not self.freeform_canvas.modo_edicao:
                    for it in self.layout_items:
                        if it["tipo"] in ("momento", "acao") and it.get("atalho", "").lower() == texto_tecla:
                            self.acionar_item_codificacao(it)
                            return True
            elif self.tabs.currentIndex() == 1:
                if key == Qt.Key.Key_Space:
                    self.alternar_play_pause_studio()
                    return True
                elif key in (Qt.Key.Key_Backspace, Qt.Key.Key_Delete):
                    if self.canvas_studio.efeito_rascunho is not None:
                        self.descartar_efeito_rascunho()
                        return True
                    kf = self.obter_keyframe_studio_ativo()
                    idx_s = self.canvas_studio.efeito_selecionado_idx
                    if kf is not None and idx_s is not None and 0 <= idx_s < len(kf.get("efeitos", [])):
                        kf["efeitos"].pop(idx_s)
                        self.canvas_studio.efeito_selecionado_idx = None
                        self.atualizar_lista_keyframes_ui()
                        self.atualizar_preview_estudio()
                        return True

        return super().eventFilter(obj, event)

    def construir_barra_superior(self, parent_layout):
        header = QFrame()
        header.setObjectName("Header")
        header.setFixedHeight(54)
        h_layout = QHBoxLayout(header)
        h_layout.setContentsMargins(14, 4, 16, 4)

        self.logo_widget = VectorLogoWidget()
        h_layout.addWidget(self.logo_widget)

        h_layout.addSpacing(12)
        h_layout.addWidget(QLabel("<span style='color:#94A3B8; font-size:11px; font-weight:bold;'>MODALIDADE:</span>"))
        self.combo_modalidade = QComboBox()
        self.combo_modalidade.addItem("Futebol", "futebol")
        self.combo_modalidade.addItem("Voleibol", "voleibol")
        self.combo_modalidade.addItem("Basquetebol", "basquetebol")
        self.combo_modalidade.addItem("Andebol", "andebol")
        self.combo_modalidade.currentIndexChanged.connect(self.ao_alterar_modalidade)
        h_layout.addWidget(self.combo_modalidade)

        h_layout.addStretch()

        btn_open = QPushButton("Abrir Vídeo")
        btn_open.clicked.connect(self.escolher_ficheiro_video)
        h_layout.addWidget(btn_open)

        btn_sessao = QPushButton("Sessão")
        menu_sessao = QMenu(self)
        act_guardar = menu_sessao.addAction("Guardar Sessão")
        act_guardar.triggered.connect(self.guardar_projeto_json)
        act_importar = menu_sessao.addAction("Importar Sessão")
        act_importar.triggered.connect(self.abrir_projeto_json)
        menu_sessao.addSeparator()
        act_recentes = menu_sessao.addAction("Projetos Recentes")
        act_recentes.triggered.connect(self.abrir_projetos_recentes_dialog)
        act_pasta = menu_sessao.addAction("Pasta Projetos")
        act_pasta.triggered.connect(self.abrir_pasta_kronos)
        btn_sessao.setMenu(menu_sessao)
        h_layout.addWidget(btn_sessao)

        self.btn_modo_livre = QPushButton("Personalizar Match Desk")
        self.btn_modo_livre.setObjectName("AccentBtn")
        self.btn_modo_livre.clicked.connect(self.alternar_modo_edicao_canvas)
        h_layout.addWidget(self.btn_modo_livre)

        btn_mais = QPushButton("Mais")
        menu_mais = QMenu(self)
        act_opt = menu_mais.addAction("Otimizar Vídeo")
        act_opt.triggered.connect(self.otimizar_video_integrado)
        menu_mais.addSeparator()
        act_ajuda = menu_mais.addAction("Ajuda / Suporte")
        act_ajuda.triggered.connect(self.abrir_menu_suporte)
        btn_mais.setMenu(menu_mais)
        h_layout.addWidget(btn_mais)

        parent_layout.addWidget(header)

    def ao_alterar_modalidade(self, idx):
        if not hasattr(self, "combo_modalidade"):
            return
        nova_mod = self.combo_modalidade.itemData(idx)
        if not nova_mod or nova_mod == getattr(self, "modalidade_atual", "futebol"):
            return

        if hasattr(self, "modalidade_atual") and hasattr(self, "presets_modalidades"):
            self.presets_modalidades[self.modalidade_atual] = copy.deepcopy(self.layout_items)

        self.modalidade_atual = nova_mod
        self.momento_ativo = None
        self.acoes_em_gravacao.clear()
        self.labels_ativas.clear()
        self.coord_campo_atual = (0.5, 0.5)
        zonas_padrao = {
            "voleibol": "Zona 6 (Fora 3m)",
            "basquetebol": "Topo / Chave",
            "andebol": "Central / Pivot",
            "futebol": "Zona 8 (Corredor Central)"
        }
        self.zona_campo_atual = zonas_padrao.get(nova_mod, "Zona 8 (Corredor Central)")

        if nova_mod in self.presets_modalidades:
            self.layout_items = copy.deepcopy(self.presets_modalidades[nova_mod])
        self.campo_oculto = not any(it.get("tipo") == "minicampo" for it in self.layout_items)

        if hasattr(self, "freeform_canvas"):
            self.freeform_canvas.widget_selecionado = None
            self.freeform_canvas.reconstruir_widgets()
        self.sincronizar_estruturas_desde_canvas()
        if self.tabs.currentIndex() == 2:
            self.desenhar_heatmap_espacial()

    @staticmethod
    def _calcular_zona_voleibol(nx, ny):
        col_6 = min(5, int(max(0.0, min(0.9999, nx)) * 6.0))
        row_3 = min(2, int(max(0.0, min(0.9999, ny)) * 3.0))

        layout_6x3 = [
            [5, 7, 4, 2, 9, 1],
            [6, 8, 3, 3, 8, 6],
            [1, 9, 2, 4, 7, 5],
        ]
        zona_num = layout_6x3[row_3][col_6]
        return f"Zona {zona_num}", f"Zona {zona_num}"

    @staticmethod
    def _calcular_zona_basquetebol(nx, ny):
        # Devolve sempre uma das etiquetas exatas do preset de basquetebol:
        # "Garrafão", "Ala / 3P" ou "Topo / Chave"
        na_area = (nx < 0.22 or nx > 0.78) and (0.30 <= ny <= 0.70)
        if na_area:
            return "Garrafão", "Garrafão"
        if ny < 0.28 or ny > 0.72:
            return "Ala / 3P", "Ala / 3P"
        return "Topo / Chave", "Topo / Chave"

    @staticmethod
    def _calcular_zona_andebol(nx, ny):
        # Espelho dividido pela linha do meio-campo (nx = 0.5):
        # Metade esquerda (nx < 0.5): Ponta Esq -> Lateral Esq -> Central / Pivot -> Lateral Dir -> Ponta Dir
        # Metade direita (nx >= 0.5): espelhado (Ponta Dir -> Lateral Dir -> Central / Pivot -> Lateral Esq -> Ponta Esq)
        if nx < 0.5:
            if ny < 0.18:
                zona_btn = "Ponta Esq"
            elif ny < 0.36:
                zona_btn = "Lateral Esq"
            elif ny > 0.82:
                zona_btn = "Ponta Dir"
            elif ny > 0.64:
                zona_btn = "Lateral Dir"
            else:
                zona_btn = "Central / Pivot"
        else:
            if ny < 0.18:
                zona_btn = "Ponta Dir"
            elif ny < 0.36:
                zona_btn = "Lateral Dir"
            elif ny > 0.82:
                zona_btn = "Ponta Esq"
            elif ny > 0.64:
                zona_btn = "Lateral Esq"
            else:
                zona_btn = "Central / Pivot"

        return zona_btn, zona_btn

    @staticmethod
    def _desenhar_campo_por_modalidade(p, mod, rect_f, mostrar_zonas=True):
        x0, y0, w, h = rect_f.x(), rect_f.y(), rect_f.width(), rect_f.height()
        x1, y1 = x0 + w, y0 + h
        cx, cy = x0 + w / 2.0, y0 + h / 2.0

        p.save()
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        p.setClipRect(rect_f.adjusted(-2.0, -2.0, 2.0, 2.0))

        if mod == "voleibol":
            row_h = h / 3.0
            col_w = w / 6.0

            # Fundo geral fora dos 3m (azul-marinho profundo) e faixas dos 3m junto à rede
            p.fillRect(rect_f, QColor("#0F294A"))
            p.fillRect(QRectF(x0 + 2.0 * col_w, y0, col_w, h), QColor("#134E6F"))
            p.fillRect(QRectF(x0 + 3.0 * col_w, y0, col_w, h), QColor("#134E6F"))

            # Grelha de zonas (linhas horizontais e divisórias verticais, sem números nas células)
            if mostrar_zonas:
                p.setPen(QPen(QColor(148, 163, 184, 110), 1.0, Qt.PenStyle.DashLine))
                for r in range(1, 3):
                    zy = y0 + row_h * r
                    p.drawLine(QPointF(x0, zy), QPointF(x1, zy))
                for c_div in [1, 2, 4, 5]:
                    zx = x0 + c_div * col_w
                    p.drawLine(QPointF(zx, y0), QPointF(zx, y1))

            # Linhas dos 3 metros junto à rede em cada lado
            p.setPen(QPen(QColor("#38BDF8"), 1.6))
            p.drawLine(QPointF(x0 + 2.0 * col_w, y0), QPointF(x0 + 2.0 * col_w, y1))
            p.drawLine(QPointF(x0 + 4.0 * col_w, y0), QPointF(x0 + 4.0 * col_w, y1))

            # Contorno exterior e rede central
            p.setPen(QPen(QColor("#E2E8F0"), 1.6))
            p.drawRect(rect_f)

            p.setPen(QPen(QColor("#F59E0B"), 2.4))
            p.drawLine(QPointF(cx, y0 - 2.0), QPointF(cx, y1 + 2.0))

        elif mod == "basquetebol":
            p.fillRect(rect_f, QColor("#2A170E"))
            p.setPen(QPen(QColor(255, 255, 255, 18), 1.0))
            n_tabuas = max(8, int(h // 8))
            for i_t in range(1, n_tabuas):
                yt = y0 + (h / n_tabuas) * i_t
                p.drawLine(QPointF(x0, yt), QPointF(x1, yt))

            ga_w = w * (5.80 / 28.0)
            ga_h = h * (4.90 / 15.0)
            ga_y0 = cy - ga_h / 2.0
            p.fillRect(QRectF(x0, ga_y0, ga_w, ga_h), QColor(2, 132, 199, 95))
            p.fillRect(QRectF(x1 - ga_w, ga_y0, ga_w, ga_h), QColor(2, 132, 199, 95))

            r_circ_x = max(12.0, w * (1.80 / 28.0))
            r_circ_y = max(12.0, h * (1.80 / 15.0))
            p.setBrush(QBrush(QColor(56, 189, 248, 35)))
            p.setPen(QPen(QColor("#E2E8F0"), 1.5))
            p.drawEllipse(QPointF(cx, cy), r_circ_x, r_circ_y)

            p.setBrush(Qt.BrushStyle.NoBrush)
            p.setPen(QPen(QColor("#F8FAFC"), 1.6))
            p.drawRect(rect_f)
            p.drawLine(QPointF(cx, y0), QPointF(cx, y1))

            p.drawRect(QRectF(x0, ga_y0, ga_w, ga_h))
            p.drawRect(QRectF(x1 - ga_w, ga_y0, ga_w, ga_h))
            p.drawEllipse(QPointF(x0 + ga_w, cy), r_circ_x, r_circ_y)
            p.drawEllipse(QPointF(x1 - ga_w, cy), r_circ_x, r_circ_y)

            tab_x_e = x0 + w * (1.20 / 28.0)
            tab_x_d = x1 - w * (1.20 / 28.0)
            tab_h = h * (1.80 / 15.0)
            p.setPen(QPen(QColor("#FFFFFF"), 2.0))
            p.drawLine(QPointF(tab_x_e, cy - tab_h / 2.0), QPointF(tab_x_e, cy + tab_h / 2.0))
            p.drawLine(QPointF(tab_x_d, cy - tab_h / 2.0), QPointF(tab_x_d, cy + tab_h / 2.0))

            cesto_x_e = x0 + w * (1.575 / 28.0)
            cesto_x_d = x1 - w * (1.575 / 28.0)
            p.setPen(QPen(QColor("#F59E0B"), 2.0))
            p.drawEllipse(QPointF(cesto_x_e, cy), 3.5, 3.5)
            p.drawEllipse(QPointF(cesto_x_d, cy), 3.5, 3.5)

            canto_y = h * (0.90 / 15.0)
            rx_3p = w * (7.20 / 28.0)
            ry_3p = h * (6.75 / 15.0)
            y_3p_sup = y0 + canto_y
            y_3p_inf = y1 - canto_y
            dy_c = cy - y_3p_sup
            dx_arco = math.sqrt(max(0.0, 1.0 - (dy_c / max(1.0, ry_3p)) ** 2)) * rx_3p
            ang_deg = math.degrees(math.atan2(dy_c, max(1.0, dx_arco)))

            p.setPen(QPen(QColor("#38BDF8"), 1.6))
            path_e = QPainterPath(QPointF(x0, y_3p_sup))
            path_e.lineTo(QPointF(cesto_x_e + dx_arco, y_3p_sup))
            path_e.arcTo(QRectF(cesto_x_e - rx_3p, cy - ry_3p, 2.0 * rx_3p, 2.0 * ry_3p), ang_deg, -2.0 * ang_deg)
            path_e.lineTo(QPointF(x0, y_3p_inf))
            p.drawPath(path_e)

            path_d = QPainterPath(QPointF(x1, y_3p_sup))
            path_d.lineTo(QPointF(cesto_x_d - dx_arco, y_3p_sup))
            path_d.arcTo(QRectF(cesto_x_d - rx_3p, cy - ry_3p, 2.0 * rx_3p, 2.0 * ry_3p), 180.0 - ang_deg, 2.0 * ang_deg)
            path_d.lineTo(QPointF(x1, y_3p_inf))
            p.drawPath(path_d)

        elif mod == "andebol":
            p.fillRect(rect_f, QColor("#122B3A"))
            p.setPen(QPen(QColor("#F8FAFC"), 1.6))
            p.drawRect(rect_f)

            p.drawLine(QPointF(cx, y0), QPointF(cx, y1))
            r_centro = h * (2.0 / 20.0)
            p.drawEllipse(QPointF(cx, cy), r_centro, r_centro)

            bal_h = h * (3.0 / 20.0)
            poste_sup = cy - bal_h / 2.0
            poste_inf = cy + bal_h / 2.0
            p.setPen(QPen(QColor("#38BDF8"), 2.4))
            p.drawLine(QPointF(x0, poste_sup), QPointF(x0, poste_inf))
            p.drawLine(QPointF(x1, poste_sup), QPointF(x1, poste_inf))

            p.save()
            p.setClipRect(rect_f, Qt.ClipOperation.IntersectClip)

            reto_6m_x = w * (6.0 / 40.0)
            r6_x = w * (6.0 / 40.0)
            r6_y = h * (6.0 / 20.0)
            p.setPen(QPen(QColor("#FFFFFF"), 1.6))

            path_6m_e = QPainterPath(QPointF(x0, poste_sup - r6_y))
            path_6m_e.arcTo(QRectF(x0 - r6_x, poste_sup - r6_y, 2.0 * r6_x, 2.0 * r6_y), 90.0, -90.0)
            path_6m_e.lineTo(QPointF(x0 + reto_6m_x, poste_inf))
            path_6m_e.arcTo(QRectF(x0 - r6_x, poste_inf - r6_y, 2.0 * r6_x, 2.0 * r6_y), 0.0, -90.0)
            p.drawPath(path_6m_e)

            path_6m_d = QPainterPath(QPointF(x1, poste_sup - r6_y))
            path_6m_d.arcTo(QRectF(x1 - r6_x, poste_sup - r6_y, 2.0 * r6_x, 2.0 * r6_y), 90.0, 90.0)
            path_6m_d.lineTo(QPointF(x1 - reto_6m_x, poste_inf))
            path_6m_d.arcTo(QRectF(x1 - r6_x, poste_inf - r6_y, 2.0 * r6_x, 2.0 * r6_y), 180.0, 90.0)
            p.drawPath(path_6m_d)

            reto_9m_x = w * (9.0 / 40.0)
            r9_x = w * (9.0 / 40.0)
            r9_y = h * (9.0 / 20.0)
            p.setPen(QPen(QColor("#F59E0B"), 1.4, Qt.PenStyle.DashLine))

            path_9m_e = QPainterPath(QPointF(x0, poste_sup - r9_y))
            path_9m_e.arcTo(QRectF(x0 - r9_x, poste_sup - r9_y, 2.0 * r9_x, 2.0 * r9_y), 90.0, -90.0)
            path_9m_e.lineTo(QPointF(x0 + reto_9m_x, poste_inf))
            path_9m_e.arcTo(QRectF(x0 - r9_x, poste_inf - r9_y, 2.0 * r9_x, 2.0 * r9_y), 0.0, -90.0)
            p.drawPath(path_9m_e)

            path_9m_d = QPainterPath(QPointF(x1, poste_sup - r9_y))
            path_9m_d.arcTo(QRectF(x1 - r9_x, poste_sup - r9_y, 2.0 * r9_x, 2.0 * r9_y), 90.0, 90.0)
            path_9m_d.lineTo(QPointF(x1 - reto_9m_x, poste_inf))
            path_9m_d.arcTo(QRectF(x1 - r9_x, poste_inf - r9_y, 2.0 * r9_x, 2.0 * r9_y), 180.0, 90.0)
            p.drawPath(path_9m_d)

            p.restore()

            l7_x_e = x0 + w * (7.0 / 40.0)
            l7_x_d = x1 - w * (7.0 / 40.0)
            l7_meio = max(3.0, h * (0.5 / 20.0))
            p.setPen(QPen(QColor("#FDE047"), 1.6))
            p.drawLine(QPointF(l7_x_e, cy - l7_meio), QPointF(l7_x_e, cy + l7_meio))
            p.drawLine(QPointF(l7_x_d, cy - l7_meio), QPointF(l7_x_d, cy + l7_meio))

            l4_x_e = x0 + w * (4.0 / 40.0)
            l4_x_d = x1 - w * (4.0 / 40.0)
            p.drawLine(QPointF(l4_x_e, cy - 2.0), QPointF(l4_x_e, cy + 2.0))
            p.drawLine(QPointF(l4_x_d, cy - 2.0), QPointF(l4_x_d, cy + 2.0))

        else:
            p.fillRect(rect_f, QColor("#042217"))
            if mostrar_zonas:
                p.setPen(QPen(QColor("#059669"), 1.0, Qt.PenStyle.DashLine))
                for c in range(1, 6):
                    zx = x0 + (w / 6.0) * c
                    p.drawLine(QPointF(zx, y0), QPointF(zx, y1))
                for r in range(1, 3):
                    zy = y0 + (h / 3.0) * r
                    p.drawLine(QPointF(x0, zy), QPointF(x1, zy))

            p.setPen(QPen(QColor("#E2E8F0"), 1.4))
            p.drawRect(rect_f)
            p.drawLine(QPointF(cx, y0), QPointF(cx, y1))
            raio_c = min(w, h) * 0.14
            p.drawEllipse(QPointF(cx, cy), raio_c, raio_c)

            # Grandes áreas e pequenas áreas
            ga_w = w * 0.155
            ga_h = h * 0.58
            ga_y0 = cy - ga_h / 2.0
            p.drawRect(QRectF(x0, ga_y0, ga_w, ga_h))
            p.drawRect(QRectF(x1 - ga_w, ga_y0, ga_w, ga_h))

            pa_w = w * 0.055
            pa_h = h * 0.27
            pa_y0 = cy - pa_h / 2.0
            p.drawRect(QRectF(x0, pa_y0, pa_w, pa_h))
            p.drawRect(QRectF(x1 - pa_w, pa_y0, pa_w, pa_h))

            # Marcas de penálti e meias-luas das grandes áreas
            pen_x_e = x0 + w * 0.105
            pen_x_d = x1 - w * 0.105
            p.setBrush(QBrush(QColor("#E2E8F0")))
            p.drawEllipse(QPointF(pen_x_e, cy), 1.6, 1.6)
            p.drawEllipse(QPointF(pen_x_d, cy), 1.6, 1.6)
            p.setBrush(Qt.BrushStyle.NoBrush)

            rx_ml = raio_c
            ry_ml = raio_c
            dx_ml_e = (x0 + ga_w) - pen_x_e
            if rx_ml > dx_ml_e > 0:
                ang_ml = math.degrees(math.acos(dx_ml_e / rx_ml))
                p.drawArc(QRectF(pen_x_e - rx_ml, cy - ry_ml, 2.0 * rx_ml, 2.0 * ry_ml), int(-ang_ml * 16), int(2.0 * ang_ml * 16))
                p.drawArc(QRectF(pen_x_d - rx_ml, cy - ry_ml, 2.0 * rx_ml, 2.0 * ry_ml), int((180.0 - ang_ml) * 16), int(2.0 * ang_ml * 16))

            # Balizas táticas nas linhas de fundo (proporcionais e discretas)
            bal_h = max(12.0, min(h * 0.14, w * 0.065))
            bal_w = max(5.0, min(w * 0.028, 14.0))
            bal_y0 = cy - bal_h / 2.0
            rect_bal_esq = QRectF(x0 - bal_w, bal_y0, bal_w, bal_h)
            rect_bal_dir = QRectF(x1, bal_y0, bal_w, bal_h)

            p.setClipRect(rect_f.adjusted(-bal_w - 4.0, -4.0, bal_w + 4.0, 4.0))
            p.setBrush(QBrush(QColor(56, 189, 248, 32)))
            p.setPen(QPen(QColor("#E2E8F0"), 1.5))
            p.drawRoundedRect(rect_bal_esq, 2.0, 2.0)
            p.drawRoundedRect(rect_bal_dir, 2.0, 2.0)

            # Rede suave no interior das balizas
            p.setPen(QPen(QColor(226, 232, 240, 85), 0.9))
            passos_rede = max(2, int(bal_h // 5))
            for i_r in range(1, passos_rede):
                ry = bal_y0 + (bal_h / passos_rede) * i_r
                p.drawLine(QPointF(x0 - bal_w + 1.0, ry), QPointF(x0, ry))
                p.drawLine(QPointF(x1, ry), QPointF(x1 + bal_w - 1.0, ry))

            # Linha da baliza destacada em azul-ciano (identidade visual KRONOS PRO)
            p.setPen(QPen(QColor("#38BDF8"), 2.4, Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
            p.drawLine(QPointF(x0, bal_y0), QPointF(x0, bal_y0 + bal_h))
            p.drawLine(QPointF(x1, bal_y0), QPointF(x1, bal_y0 + bal_h))

        p.restore()

    @staticmethod
    def gerar_icone_app_k(tamanho=128):
        pix = QPixmap(tamanho, tamanho)
        pix.fill(Qt.GlobalColor.transparent)
        p = QPainter(pix)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)

        ox, oy = tamanho * 0.08, tamanho * 0.08
        tam = tamanho * 0.84

        grad = QLinearGradient(ox, oy, ox + tam, oy + tam)
        grad.setColorAt(0.0, QColor("#38BDF8"))
        grad.setColorAt(0.5, QColor("#0284C7"))
        grad.setColorAt(1.0, QColor("#0F172A"))

        escudo = QPolygonF([
            QPointF(ox + tam * 0.50, oy + tam * 0.03),
            QPointF(ox + tam * 0.95, oy + tam * 0.26),
            QPointF(ox + tam * 0.84, oy + tam * 0.95),
            QPointF(ox + tam * 0.50, oy + tam * 0.81),
            QPointF(ox + tam * 0.16, oy + tam * 0.95),
            QPointF(ox + tam * 0.05, oy + tam * 0.26)
        ])
        p.setBrush(QBrush(grad))
        p.setPen(QPen(QColor("#7DD3FC"), max(1.5, tamanho * 0.035)))
        p.drawPolygon(escudo)

        p.setPen(QPen(QColor("#F59E0B"), max(2.2, tamanho * 0.065), Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
        p.drawLine(QPointF(ox + tam * 0.34, oy + tam * 0.27), QPointF(ox + tam * 0.34, oy + tam * 0.69))

        p.setPen(QPen(QColor("#FFFFFF"), max(2.0, tamanho * 0.055), Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
        p.drawLine(QPointF(ox + tam * 0.68, oy + tam * 0.27), QPointF(ox + tam * 0.41, oy + tam * 0.48))
        p.drawLine(QPointF(ox + tam * 0.41, oy + tam * 0.48), QPointF(ox + tam * 0.68, oy + tam * 0.69))

        p.end()
        return QIcon(pix)

    def abrir_menu_suporte(self):
        status, dias_rest, est = self.lic_mgr.verificar_validade()
        email_lic = est.get("email", "Não ativado") if est else "Não ativado"
        info_diag = (
            f"Aplicação: {APP_NAME} v{APP_VERSION}\n"
            f"Modalidade: {getattr(self, 'modalidade_atual', 'futebol')}\n"
            f"Sistema: {platform.system()} {platform.release()} ({platform.machine()})\n"
            f"Licença: {email_lic} (Estado: {status}, {dias_rest} dias offline restantes)\n"
            f"Vídeo ativo: {os.path.basename(self.caminho_video) if self.caminho_video else 'Nenhum'}"
        )
        msg = QMessageBox(self)
        msg.setWindowTitle(f"Acerca e Suporte — {APP_NAME}")
        msg.setText(
            f"<b>{APP_NAME} v{APP_VERSION}</b> — {APP_SUBTITLE}<br><br>"
            f"<b>Privacidade:</b> Todos os vídeos e dados táticos ficam exclusivamente neste computador.<br><br>"
            f"<pre style='background:#0B0B12; padding:8px; color:#38BDF8;'>{info_diag}</pre>"
        )
        btn_copiar = msg.addButton("Gerar e Copiar Relatório de Suporte", QMessageBox.ButtonRole.ActionRole)
        msg.addButton("Fechar", QMessageBox.ButtonRole.AcceptRole)
        msg.exec()
        if msg.clickedButton() == btn_copiar:
            QApplication.clipboard().setText(info_diag)
            QMessageBox.information(self, "Copiado", "Relatório técnico de suporte copiado para a área de transferência.")

    def otimizar_video_integrado(self):
        if not os.path.exists(self.caminho_video):
            QMessageBox.warning(self, "Aviso", "Selecione primeiro um ficheiro de vídeo.")
            return
        self.media_player.pause()
        self.btn_play.definir_modo("play")
        self.dlg_opt = QProgressDialog("A preparar otimização MP4 (H.264)...", None, 0, 0, self)
        self.dlg_opt.setWindowTitle("Otimizador Integrado KRONOS")
        self.dlg_opt.setWindowModality(Qt.WindowModality.WindowModal)
        self.dlg_opt.show()

        self.worker_opt = VideoOptimizeWorker(self.caminho_video)
        self.worker_opt.progresso.connect(self.dlg_opt.setLabelText)
        self.worker_opt.concluido.connect(self.ao_concluir_otimizacao)
        self.worker_opt.erro.connect(lambda e: (self.dlg_opt.close(), QMessageBox.critical(self, "Erro", e)))
        self.worker_opt.start()

    def ao_concluir_otimizacao(self, novo_caminho):
        self.dlg_opt.close()
        self.caminho_video = novo_caminho
        self.iniciar_video()
        QMessageBox.information(self, "Vídeo Otimizado", f"Vídeo convertido e carregado com sucesso:\n{novo_caminho}")

    def alternar_modo_edicao_canvas(self):
        if not hasattr(self, "freeform_canvas") or self.freeform_canvas is None:
            return
        ativo = not self.freeform_canvas.modo_edicao
        self.freeform_canvas.modo_edicao = ativo
        if hasattr(self, "barra_ferramentas_canvas") and self.barra_ferramentas_canvas is not None:
            self.barra_ferramentas_canvas.setVisible(ativo)
        if ativo:
            self.btn_modo_livre.setText("Concluir Edição do Match Desk")
            self.btn_modo_livre.setObjectName("WarnBtn")
        else:
            self.btn_modo_livre.setText("Personalizar Match Desk")
            self.btn_modo_livre.setObjectName("AccentBtn")
            self.freeform_canvas.widget_selecionado = None
        estilo = self.btn_modo_livre.style()
        if estilo is not None:
            estilo.unpolish(self.btn_modo_livre)
            estilo.polish(self.btn_modo_livre)
        self.btn_modo_livre.update()
        self.freeform_canvas.atualizar_visual_todos()

    def alternar_visibilidade_codigo(self):
        visivel = self.painel_esquerdo.isVisible()
        self.painel_esquerdo.setVisible(not visivel)
        self.aba_expandir_esq.setVisible(visivel)

    def alternar_visibilidade_grupos(self):
        visivel = self.painel_direito.isVisible()
        self.painel_direito.setVisible(not visivel)
        self.aba_expandir_dir.setVisible(visivel)

    def atalho_avancar_l(self):
        if self.media_player.playbackState() != QMediaPlayer.PlaybackState.PlayingState:
            self.alternar_play_pause()
        else:
            self.ciclar_velocidade()

    def configurar_tab_codificacao(self):
        outer_h = QHBoxLayout(self.tab_codificacao)
        outer_h.setContentsMargins(4, 4, 4, 4)
        outer_h.setSpacing(2)

        self.aba_expandir_esq = QPushButton("▶\nC\nÓ\nD\nI\nG\nO")
        self.aba_expandir_esq.setObjectName("CollapseBtn")
        self.aba_expandir_esq.setFixedWidth(24)
        self.aba_expandir_esq.setSizePolicy(QSizePolicy.Policy.Fixed, QSizePolicy.Policy.Expanding)
        self.aba_expandir_esq.clicked.connect(self.alternar_visibilidade_codigo)
        self.aba_expandir_esq.setVisible(False)
        outer_h.addWidget(self.aba_expandir_esq)

        self.splitter_main = QSplitter(Qt.Orientation.Horizontal)
        self.splitter_main.setChildrenCollapsible(True)
        self.splitter_main.setCollapsible(0, True)
        self.splitter_main.setCollapsible(1, False)
        self.splitter_main.setCollapsible(2, False)
        self.splitter_main.setHandleWidth(10)
        outer_h.addWidget(self.splitter_main, 1)

        self.aba_expandir_dir = QPushButton("◀\nG\nR\nU\nP\nO\nS")
        self.aba_expandir_dir.setObjectName("CollapseBtn")
        self.aba_expandir_dir.setFixedWidth(24)
        self.aba_expandir_dir.setSizePolicy(QSizePolicy.Policy.Fixed, QSizePolicy.Policy.Expanding)
        self.aba_expandir_dir.clicked.connect(self.alternar_visibilidade_grupos)
        self.aba_expandir_dir.setVisible(False)
        outer_h.addWidget(self.aba_expandir_dir)

        self.painel_esquerdo = QFrame()
        self.painel_esquerdo.setObjectName("Panel")
        self.painel_esquerdo.setMinimumWidth(140)
        v_esq = QVBoxLayout(self.painel_esquerdo)
        v_esq.setContentsMargins(0, 0, 0, 6)
        v_esq.setSpacing(4)

        sub_esq = QFrame()
        sub_esq.setObjectName("SubHeader")
        h_sub_esq = QHBoxLayout(sub_esq)
        h_sub_esq.setContentsMargins(10, 6, 8, 6)
        h_sub_esq.addWidget(QLabel("<b style='color:#38BDF8; letter-spacing:0.6px;'>JANELA DE CÓDIGO</b>"))
        h_sub_esq.addStretch()
        btn_hide_esq = QPushButton("Ocultar")
        btn_hide_esq.setObjectName("CollapseBtn")
        btn_hide_esq.clicked.connect(self.alternar_visibilidade_codigo)
        h_sub_esq.addWidget(btn_hide_esq)
        v_esq.addWidget(sub_esq)

        lbl_hint_mac = QLabel("<span style='color:#71717A; font-size:10px;'>Dica: Clique Direito num botão para definir Pré/Pós ou Ao Clique.</span>")
        lbl_hint_mac.setWordWrap(True)
        lbl_hint_mac.setContentsMargins(10, 2, 10, 0)
        v_esq.addWidget(lbl_hint_mac)

        self.barra_ferramentas_canvas = QFrame()
        v_add = QVBoxLayout(self.barra_ferramentas_canvas)
        v_add.setContentsMargins(8, 2, 8, 4)
        v_add.setSpacing(4)

        h_add_1 = QHBoxLayout()
        h_add_1.setSpacing(4)
        b_add_m = QPushButton("+ Momento")
        b_add_m.clicked.connect(lambda: self.freeform_canvas.adicionar_novo_item("momento"))
        b_add_a = QPushButton("+ Ação")
        b_add_a.clicked.connect(lambda: self.freeform_canvas.adicionar_novo_item("acao"))
        b_add_l = QPushButton("+ Etiqueta")
        b_add_l.clicked.connect(lambda: self.freeform_canvas.adicionar_novo_item("label"))
        h_add_1.addWidget(b_add_m)
        h_add_1.addWidget(b_add_a)
        h_add_1.addWidget(b_add_l)
        v_add.addLayout(h_add_1)

        h_add_2 = QHBoxLayout()
        h_add_2.setSpacing(4)
        b_desc_campo = QPushButton("Descartar Campo")
        b_desc_campo.setObjectName("DangerBtn")
        b_repor_campo = QPushButton("+ Campo")
        b_repor_campo.setObjectName("SuccessBtn")
        b_undo_c = QPushButton("↶")
        b_undo_c.setToolTip("Retroceder alteração na janela de código")
        b_undo_c.clicked.connect(self.desfazer_janela_codigo)
        b_redo_c = QPushButton("↷")
        b_redo_c.setToolTip("Avançar alteração na janela de código")
        b_redo_c.clicked.connect(self.refazer_janela_codigo)
        h_add_2.addWidget(b_desc_campo)
        h_add_2.addWidget(b_repor_campo)
        h_add_2.addWidget(b_undo_c)
        h_add_2.addWidget(b_redo_c)
        v_add.addLayout(h_add_2)

        self.barra_ferramentas_canvas.setVisible(False)
        v_esq.addWidget(self.barra_ferramentas_canvas)

        self.scroll_canvas = QScrollArea()
        self.scroll_canvas.setWidgetResizable(True)
        self.scroll_canvas.setStyleSheet("border: none;")
        self.freeform_canvas = FreeformCodeCanvas(self)
        b_desc_campo.clicked.connect(self.freeform_canvas.descartar_campo)
        b_repor_campo.clicked.connect(self.freeform_canvas.repor_campo)
        self.freeform_canvas.reconstruir_widgets()
        self.scroll_canvas.setWidget(self.freeform_canvas)
        v_esq.addWidget(self.scroll_canvas, 1)

        self.splitter_main.addWidget(self.painel_esquerdo)

        self.splitter_centro = QSplitter(Qt.Orientation.Vertical)
        self.splitter_centro.setChildrenCollapsible(True)
        self.splitter_centro.setHandleWidth(10)
        self.painel_video_container = self.criar_painel_video_gpu()
        self.timeline_widget = TimelineWidget(self)

        self.splitter_centro.addWidget(self.painel_video_container)
        self.splitter_centro.addWidget(self.timeline_widget)
        self.splitter_centro.setSizes([585, 240])
        self.splitter_main.addWidget(self.splitter_centro)

        self.painel_direito = QFrame()
        self.painel_direito.setObjectName("Panel")
        self.painel_direito.setMinimumWidth(185)
        v_dir_main = QVBoxLayout(self.painel_direito)
        v_dir_main.setContentsMargins(0, 0, 0, 6)
        v_dir_main.setSpacing(4)

        sub_dir = QFrame()
        sub_dir.setObjectName("SubHeader")
        h_sub_dir = QHBoxLayout(sub_dir)
        h_sub_dir.setContentsMargins(10, 6, 8, 6)
        h_sub_dir.addWidget(QLabel("<b style='color:#38BDF8; letter-spacing:0.6px;'>GRUPOS E CONTADORES</b>"))
        h_sub_dir.addStretch()
        btn_hide_dir = QPushButton("Ocultar")
        btn_hide_dir.setObjectName("CollapseBtn")
        btn_hide_dir.clicked.connect(self.alternar_visibilidade_grupos)
        h_sub_dir.addWidget(btn_hide_dir)
        v_dir_main.addWidget(sub_dir)

        scroll_dir = QScrollArea()
        scroll_dir.setWidgetResizable(True)
        scroll_dir.setStyleSheet("border: none;")
        self.container_grupos_dinamico = QWidget()
        self.layout_grupos_dinamico = QVBoxLayout(self.container_grupos_dinamico)
        self.layout_grupos_dinamico.setContentsMargins(10, 6, 10, 10)
        self.layout_grupos_dinamico.setSpacing(5)
        scroll_dir.setWidget(self.container_grupos_dinamico)
        v_dir_main.addWidget(scroll_dir, 1)

        self.reconstruir_conteudo_painel_direito()
        self.splitter_main.addWidget(self.painel_direito)

        self.splitter_main.setSizes([295, 965, 270])

    def limpar_layout(self, layout):
        if layout is None:
            return
        while layout.count():
            item = layout.takeAt(0)
            if item is None:
                continue
            w = item.widget()
            if w is not None:
                w.setParent(None)
                w.deleteLater()
            sub_lay = item.layout()
            if sub_lay is not None:
                self.limpar_layout(sub_lay)
                sub_lay.deleteLater()

    def reconstruir_conteudo_painel_direito(self):
        self.limpar_layout(self.layout_grupos_dinamico)
        v_lay = self.layout_grupos_dinamico

        self.botoes_grupos_dash = []
        self.dash_labels = {}
        for acao, info in self.todas_pistas.items():
            row = QHBoxLayout()
            lbl_qtd = QLabel("0")
            lbl_qtd.setAlignment(Qt.AlignmentFlag.AlignCenter)
            lbl_qtd.setFixedSize(28, 22)
            lbl_qtd.setStyleSheet(f"background-color: {info['cor']}; color: white; border-radius: 4px; font-weight: bold;")
            row.addWidget(lbl_qtd)

            btn_grp = QPushButton(acao)
            btn_grp.setStyleSheet("text-align: left;")
            btn_grp.setToolTip("Clique para abrir a subjanela com todos os cortes deste grupo")
            btn_grp.clicked.connect(lambda checked, a=acao, c=info['cor']: self.abrir_subjanela_grupo(a, c))
            row.addWidget(btn_grp, 1)

            v_lay.addLayout(row)
            self.dash_labels[acao] = lbl_qtd
            self.botoes_grupos_dash.append((btn_grp, acao))

        v_lay.addSpacing(10)
        v_lay.addWidget(QLabel("<b style='color:#94A3B8;'>ANOTAÇÃO DO LANCE</b>"))
        self.entry_nota = QLineEdit()
        self.entry_nota.setPlaceholderText("Inserir observação tática...")
        v_lay.addWidget(self.entry_nota)

        btn_nota = QPushButton("Guardar Nota no Corte")
        btn_nota.clicked.connect(self.guardar_nota_corte)
        v_lay.addWidget(btn_nota)

        v_lay.addSpacing(12)
        v_lay.addWidget(QLabel("<b style='color:#94A3B8;'>MÓDULOS DE SAÍDA</b>"))

        btn_studio = QPushButton("Enviar Cortes para Vision Studio")
        btn_studio.setObjectName("AccentBtn")
        btn_studio.clicked.connect(self.importar_todos_para_estudio)
        v_lay.addWidget(btn_studio)

        btn_fast_mp4 = QPushButton("Exportação Direta Rápida (.MP4)")
        btn_fast_mp4.setObjectName("SuccessBtn")
        btn_fast_mp4.clicked.connect(lambda: self.iniciar_exportacao_mp4(modo_rapido=True))
        v_lay.addWidget(btn_fast_mp4)

        btn_matrix = QPushButton("Abrir Matriz de Cruzamento")
        btn_matrix.clicked.connect(self.abrir_janela_matrix)
        v_lay.addWidget(btn_matrix)

        btn_csv = QPushButton("Exportar Dados (CSV)")
        btn_csv.clicked.connect(self.exportar_excel)
        v_lay.addWidget(btn_csv)

        v_lay.addStretch()

        if hasattr(self, "painel_direito"):
            self.painel_direito.installEventFilter(self)
        self.atualizar_escala_grupos_contadores()

    def atualizar_escala_grupos_contadores(self):
        if not hasattr(self, "botoes_grupos_dash") or not hasattr(self, "painel_direito"):
            return
        larg_painel = max(140, self.painel_direito.width())
        tam_pt = max(8, min(13, int(round(larg_painel / 24.0))))
        for btn, texto_orig in self.botoes_grupos_dash:
            try:
                f = btn.font()
                f.setPointSize(tam_pt)
                btn.setFont(f)
                fm = btn.fontMetrics()
                larg_util = max(50, larg_painel - 85)
                btn.setText(fm.elidedText(texto_orig, Qt.TextElideMode.ElideRight, larg_util))
            except RuntimeError:
                pass

    def abrir_subjanela_grupo(self, nome_grupo, cor_hex):
        dlg = SubjanelaCortesGrupoDialog(self, nome_grupo, cor_hex)
        dlg.exec()

    def criar_painel_video_gpu(self):
        container = QFrame()
        container.setObjectName("Panel")
        container.setMinimumHeight(140)
        v_lay = QVBoxLayout(container)
        v_lay.setContentsMargins(0, 0, 0, 0)
        v_lay.setSpacing(0)

        self.video_widget = QVideoWidget()
        self.video_widget.setStyleSheet("background-color: #000000;")
        v_lay.addWidget(self.video_widget, 1)

        transport = QFrame()
        transport.setObjectName("TransportBar")
        v_trans = QVBoxLayout(transport)
        v_trans.setContentsMargins(10, 6, 10, 8)
        v_trans.setSpacing(6)

        self.slider_progresso = QSlider(Qt.Orientation.Horizontal)
        self.slider_progresso.setRange(0, 1000)
        self.slider_progresso.sliderPressed.connect(self.ao_pressionar_slider)
        self.slider_progresso.sliderReleased.connect(self.ao_soltar_slider)
        self.slider_progresso.sliderMoved.connect(self.ao_mover_slider)
        v_trans.addWidget(self.slider_progresso)

        ctrl_lay = QHBoxLayout()
        ctrl_lay.setSpacing(8)

        btn_load_inline = QPushButton("Carregar Vídeo")
        btn_load_inline.setObjectName("AccentBtn")
        btn_load_inline.clicked.connect(self.iniciar_video)
        ctrl_lay.addWidget(btn_load_inline)

        self.btn_recuar_15 = VectorTransportButton("back15")
        self.btn_recuar_15.setToolTip("Retroceder 15 segundos (Seta Esquerda)")
        self.btn_recuar_15.clicked.connect(lambda: self.saltar_segundos(-15.0))
        ctrl_lay.addWidget(self.btn_recuar_15)

        self.btn_play = VectorTransportButton("play")
        self.btn_play.setToolTip("Reproduzir / Pausa (Barra de Espaço)")
        self.btn_play.clicked.connect(self.alternar_play_pause)
        ctrl_lay.addWidget(self.btn_play)

        self.btn_avancar_15 = VectorTransportButton("fwd15")
        self.btn_avancar_15.setToolTip("Avançar 15 segundos (Seta Direita)")
        self.btn_avancar_15.clicked.connect(lambda: self.saltar_segundos(15.0))
        ctrl_lay.addWidget(self.btn_avancar_15)

        self.lbl_tempo_atual = QLabel("00:00.00")
        self.lbl_tempo_atual.setStyleSheet(
            "color: #F59E0B; font-family: 'SF Mono', 'Menlo', 'Consolas', monospace; "
            "font-size: 15px; font-weight: bold; margin-left: 8px;"
        )
        ctrl_lay.addWidget(self.lbl_tempo_atual)

        lbl_barra = QLabel("/")
        lbl_barra.setStyleSheet("color: #52525B; font-size: 14px; font-weight: bold;")
        ctrl_lay.addWidget(lbl_barra)

        self.lbl_tempo_total = QLabel("00:00")
        self.lbl_tempo_total.setStyleSheet(
            "color: #A1A1AA; font-family: 'SF Mono', 'Menlo', 'Consolas', monospace; "
            "font-size: 14px; font-weight: bold;"
        )
        ctrl_lay.addWidget(self.lbl_tempo_total)

        ctrl_lay.addStretch()

        ctrl_lay.addWidget(QLabel("Velocidade:"))
        self.combo_velocidade = QComboBox()
        self.combo_velocidade.setMinimumWidth(102)
        for v in self.lista_velocidades:
            self.combo_velocidade.addItem(f"{v:.2f}x", v)
        self.combo_velocidade.setCurrentIndex(2)
        self.combo_velocidade.currentIndexChanged.connect(self.ao_escolher_velocidade_combo)
        ctrl_lay.addWidget(self.combo_velocidade)

        ctrl_lay.addSpacing(6)
        ctrl_lay.addWidget(QLabel("Zoom Timeline:"))
        self.slider_zoom = QSlider(Qt.Orientation.Horizontal)
        self.slider_zoom.setRange(10, 240)
        self.slider_zoom.setValue(45)
        self.slider_zoom.setFixedWidth(95)
        self.slider_zoom.valueChanged.connect(self.alterar_zoom_timeline)
        ctrl_lay.addWidget(self.slider_zoom)

        v_trans.addLayout(ctrl_lay)
        v_lay.addWidget(transport)
        return container

    def configurar_tab_estudio(self):
        layout = QVBoxLayout(self.tab_estudio)
        layout.setContentsMargins(6, 6, 6, 6)
        splitter = QSplitter(Qt.Orientation.Horizontal)
        splitter.setChildrenCollapsible(False)
        splitter.setHandleWidth(10)
        layout.addWidget(splitter)

        f_esq = QFrame()
        f_esq.setObjectName("Panel")
        f_esq.setMinimumWidth(210)
        v_esq = QVBoxLayout(f_esq)
        v_esq.setContentsMargins(0, 0, 0, 8)
        v_esq.setSpacing(6)

        sub_s1 = QFrame()
        sub_s1.setObjectName("SubHeader")
        h_s1 = QHBoxLayout(sub_s1)
        h_s1.setContentsMargins(10, 6, 10, 6)
        h_s1.addWidget(QLabel("<b style='color:#38BDF8; letter-spacing:0.6px;'>1. CORTES E PAUSAS TÁTICAS</b>"))
        v_esq.addWidget(sub_s1)

        v_esq_inner = QVBoxLayout()
        v_esq_inner.setContentsMargins(8, 2, 8, 2)
        v_esq_inner.setSpacing(6)

        btn_sync = QPushButton("Sincronizar Cortes do Match Desk")
        btn_sync.setObjectName("AccentBtn")
        btn_sync.clicked.connect(self.importar_todos_para_estudio)
        v_esq_inner.addWidget(btn_sync)

        h_ord = QHBoxLayout()
        btn_up = QPushButton("▲ Subir")
        btn_up.clicked.connect(lambda: self.mover_clip_estudio(-1))
        btn_down = QPushButton("▼ Descer")
        btn_down.clicked.connect(lambda: self.mover_clip_estudio(1))
        btn_del_c = QPushButton("Remover Corte")
        btn_del_c.setObjectName("DangerBtn")
        btn_del_c.clicked.connect(self.remover_clip_estudio)
        h_ord.addWidget(btn_up)
        h_ord.addWidget(btn_down)
        h_ord.addWidget(btn_del_c)
        v_esq_inner.addLayout(h_ord)

        v_esq_inner.addWidget(QLabel("<b style='color:#94A3B8;'>CORTES DA PALESTRA:</b>"))
        self.lista_clips_studio_w = QListWidget()
        self.lista_clips_studio_w.setMaximumHeight(195)
        self.lista_clips_studio_w.currentRowChanged.connect(self.selecionar_clip_estudio)
        v_esq_inner.addWidget(self.lista_clips_studio_w)

        v_esq_inner.addWidget(QLabel("<b style='color:#F59E0B;'>PAUSAS TÁTICAS NESTE CORTE:</b>"))
        self.lista_keyframes_w = QListWidget()
        self.lista_keyframes_w.currentRowChanged.connect(self.ao_selecionar_keyframe_lista)
        v_esq_inner.addWidget(self.lista_keyframes_w, 1)

        f_kf_cfg = QFrame()
        f_kf_cfg.setStyleSheet("background-color:#0B0B14; border:1px solid #1E293B; border-radius:6px;")
        v_kf = QVBoxLayout(f_kf_cfg)
        v_kf.setContentsMargins(8, 6, 8, 6)
        v_kf.setSpacing(4)

        h_dur_kf = QHBoxLayout()
        h_dur_kf.addWidget(QLabel("Duração:"))
        self.spin_duracao_kf = QDoubleSpinBox()
        self.spin_duracao_kf.setRange(0.5, 20.0)
        self.spin_duracao_kf.setSingleStep(0.5)
        self.spin_duracao_kf.setSuffix(" s")
        self.spin_duracao_kf.setValue(3.0)
        self.spin_duracao_kf.valueChanged.connect(self.ao_mudar_duracao_keyframe)
        h_dur_kf.addWidget(self.spin_duracao_kf)

        btn_del_kf = QPushButton("Apagar Pausa")
        btn_del_kf.setObjectName("DangerBtn")
        btn_del_kf.clicked.connect(self.eliminar_keyframe_ativo)
        h_dur_kf.addWidget(btn_del_kf)
        v_kf.addLayout(h_dur_kf)

        self.edt_nota_studio = QLineEdit()
        self.edt_nota_studio.setPlaceholderText("Legenda tática desta pausa...")
        self.edt_nota_studio.textEdited.connect(self.ao_editar_nota_studio)
        v_kf.addWidget(self.edt_nota_studio)
        v_esq_inner.addWidget(f_kf_cfg)

        self.chk_mask_3d = QCheckBox("Mascaramento 3D (Jogadores por cima)")
        self.chk_mask_3d.setChecked(True)
        self.chk_mask_3d.toggled.connect(lambda: self.atualizar_preview_estudio())
        v_esq_inner.addWidget(self.chk_mask_3d)

        self.chk_title_card = QCheckBox("Quadro de Título Inicial (2s no MP4)")
        self.chk_title_card.setChecked(True)
        v_esq_inner.addWidget(self.chk_title_card)

        self.chk_minimapa = QCheckBox("Mostrar Radar Tático 2D no Canto")
        self.chk_minimapa.setChecked(True)
        self.chk_minimapa.toggled.connect(lambda: self.atualizar_preview_estudio())
        v_esq_inner.addWidget(self.chk_minimapa)

        v_esq.addLayout(v_esq_inner, 1)
        splitter.addWidget(f_esq)

        f_centro = QFrame()
        f_centro.setObjectName("Panel")
        v_centro = QVBoxLayout(f_centro)
        v_centro.setContentsMargins(6, 6, 6, 8)
        v_centro.setSpacing(6)

        h_top_visor = QHBoxLayout()
        self.lbl_estado_visor_studio = QLabel("<b style='color:#38BDF8; font-size:13px;'>MODO VÍDEO: Reproduza o corte ou passe o rato na timeline abaixo</b>")
        h_top_visor.addWidget(self.lbl_estado_visor_studio)
        h_top_visor.addStretch()

        self.btn_voltar_video_corte = QPushButton("Voltar ao Vídeo Contínuo")
        self.btn_voltar_video_corte.setObjectName("AccentBtn")
        self.btn_voltar_video_corte.setVisible(False)
        self.btn_voltar_video_corte.clicked.connect(self.ativar_modo_video_intero_studio)
        h_top_visor.addWidget(self.btn_voltar_video_corte)
        v_centro.addLayout(h_top_visor)

        self.stack_visor_studio = QStackedWidget()
        self.video_widget_studio = QVideoWidget()
        self.video_widget_studio.setStyleSheet("background-color: #000000;")
        self.stack_visor_studio.addWidget(self.video_widget_studio)

        self.canvas_studio = StudioCanvasWidget(self)
        self.stack_visor_studio.addWidget(self.canvas_studio)
        v_centro.addWidget(self.stack_visor_studio, 1)

        self.barra_teste_efeito = QFrame()
        self.barra_teste_efeito.setStyleSheet("background-color:#0F172A; border:1px solid #F59E0B; border-radius:6px;")
        h_teste = QHBoxLayout(self.barra_teste_efeito)
        h_teste.setContentsMargins(10, 4, 10, 4)
        h_teste.addWidget(QLabel("<b style='color:#FDE047;'>MODO TESTE:</b> Ajuste o efeito na imagem antes de guardar"))
        h_teste.addStretch()
        btn_cor_pol = QPushButton("Cor")
        btn_cor_pol.clicked.connect(self.escolher_cor_efeito_rascunho)
        h_teste.addWidget(btn_cor_pol)
        btn_conf_ef = QPushButton("Confirmar Efeito")
        btn_conf_ef.setObjectName("SuccessBtn")
        btn_conf_ef.clicked.connect(self.confirmar_efeito_rascunho)
        h_teste.addWidget(btn_conf_ef)
        btn_desc_ef = QPushButton("Descartar")
        btn_desc_ef.setObjectName("DangerBtn")
        btn_desc_ef.clicked.connect(self.descartar_efeito_rascunho)
        h_teste.addWidget(btn_desc_ef)
        self.barra_teste_efeito.setVisible(False)
        v_centro.addWidget(self.barra_teste_efeito)

        h_scrub = QHBoxLayout()
        self.btn_play_studio = VectorTransportButton("play")
        self.btn_play_studio.clicked.connect(self.alternar_play_pause_studio)
        h_scrub.addWidget(self.btn_play_studio)

        self.studio_nle_timeline = StudioNLETimelineWidget(self)
        h_scrub.addWidget(self.studio_nle_timeline, 1)

        self.lbl_tempo_corte_studio = QLabel("00:00.00 / 00:00.00")
        self.lbl_tempo_corte_studio.setStyleSheet("color:#F59E0B; font-family:'SF Mono'; font-size:13px; font-weight:bold;")
        h_scrub.addWidget(self.lbl_tempo_corte_studio)
        v_centro.addLayout(h_scrub)

        h_bar = QHBoxLayout()
        btn_capturar_kf = QPushButton("+ Adicionar Pausa Tática")
        btn_capturar_kf.setObjectName("WarnBtn")
        btn_capturar_kf.setFixedHeight(34)
        btn_capturar_kf.clicked.connect(self.capturar_fotograma_livetag_studio)
        h_bar.addWidget(btn_capturar_kf, 2)

        btn_auto = QPushButton("Detetar Jogadores (IA)")
        btn_auto.setObjectName("AccentBtn")
        btn_auto.setFixedHeight(34)
        btn_auto.clicked.connect(self.auto_detetar_jogadores)
        h_bar.addWidget(btn_auto, 1)

        v_centro.addLayout(h_bar)
        splitter.addWidget(f_centro)

        f_dir = QFrame()
        f_dir.setObjectName("Panel")
        f_dir.setMinimumWidth(205)
        v_dir = QVBoxLayout(f_dir)
        v_dir.setContentsMargins(0, 0, 0, 8)
        v_dir.setSpacing(6)

        sub_s3 = QFrame()
        sub_s3.setObjectName("SubHeader")
        h_s3 = QHBoxLayout(sub_s3)
        h_s3.setContentsMargins(10, 6, 10, 6)
        h_s3.addWidget(QLabel("<b style='color:#38BDF8; letter-spacing:0.6px;'>2. EFEITOS TÁTICOS</b>"))
        v_dir.addWidget(sub_s3)

        v_dir_inner = QVBoxLayout()
        v_dir_inner.setContentsMargins(8, 2, 8, 2)
        v_dir_inner.setSpacing(5)

        self.botoes_ferramentas = {}
        ferramentas = [
            ("anel", "Anel Jogador"),
            ("spotlight", "Foco Luz"),
            ("linha", "Zona / Polígono"),
            ("seta", "Seta 3D"),
            ("cone", "Cone Visão"),
            ("sombra", "Sombra Cobertura"),
            ("ghost", "Mover Atleta"),
            ("zoom", "Lupa 2.0x"),
            ("livre", "Caneta Livre"),
            ("texto", "Caixa de Texto"),
            ("laser", "Ponteiro Laser")
        ]
        for cod, nome in ferramentas:
            b = QPushButton(f"  {nome}")
            b.setIcon(self.criar_icone_efeito(cod, self.cores_efeitos.get(cod, "#00E5FF")))
            b.setIconSize(QSize(18, 18))
            b.setToolTip("Clique direito para predefinir a cor padrão deste efeito")
            b.setContextMenuPolicy(Qt.ContextMenuPolicy.CustomContextMenu)
            b.customContextMenuRequested.connect(lambda pos, c=cod: self.predefinir_cor_ferramenta(c))
            b.setStyleSheet("text-align: left; padding: 6px 10px;")
            b.clicked.connect(lambda checked, c=cod: self.selecionar_ferramenta_studio(c))
            v_dir_inner.addWidget(b)
            self.botoes_ferramentas[cod] = b
        self.selecionar_ferramenta_studio("anel")

        btn_calib = QPushButton("Calibrar Campo (4 Pts)")
        btn_calib.setToolTip("Calibração por Homografia nos 4 cantos da Grande Área (16.5m x 40.32m)")
        btn_calib.clicked.connect(self.iniciar_calibracao_homografia)
        v_dir_inner.addWidget(btn_calib)

        h_limp = QHBoxLayout()
        btn_undo = QPushButton("Anular Último")
        btn_undo.clicked.connect(self.desfazer_efeito_studio)
        h_limp.addWidget(btn_undo)

        btn_clear = QPushButton("Limpar Pausa")
        btn_clear.setObjectName("DangerBtn")
        btn_clear.clicked.connect(self.limpar_efeitos_studio)
        h_limp.addWidget(btn_clear)
        v_dir_inner.addLayout(h_limp)

        v_dir_inner.addStretch()
        v_dir_inner.addWidget(QLabel("<b style='color:#94A3B8;'>EXPORTAÇÃO DE PALESTRA</b>"))

        btn_export_pdf = QPushButton("Exportar Dossier Visual (.PDF)")
        btn_export_pdf.setObjectName("AccentBtn")
        btn_export_pdf.setFixedHeight(34)
        btn_export_pdf.clicked.connect(self.exportar_relatorio_pdf_balneario)
        v_dir_inner.addWidget(btn_export_pdf)

        btn_export_mp4 = QPushButton("Renderizar Filme c/ Pausas (.MP4)")
        btn_export_mp4.setObjectName("SuccessBtn")
        btn_export_mp4.setFixedHeight(38)
        btn_export_mp4.clicked.connect(lambda: self.iniciar_exportacao_mp4(modo_rapido=False))
        v_dir_inner.addWidget(btn_export_mp4)

        v_dir.addLayout(v_dir_inner, 1)
        splitter.addWidget(f_dir)
        splitter.setSizes([285, 985, 275])

    def configurar_tab_analytics(self):
        layout = QHBoxLayout(self.tab_analytics)
        layout.setContentsMargins(6, 6, 6, 6)
        splitter = QSplitter(Qt.Orientation.Horizontal)
        splitter.setChildrenCollapsible(True)
        splitter.setHandleWidth(8)
        layout.addWidget(splitter)

        f_esq = QFrame()
        f_esq.setObjectName("Panel")
        f_esq.setMinimumWidth(230)
        v_esq = QVBoxLayout(f_esq)
        v_esq.setContentsMargins(10, 10, 10, 10)
        v_esq.setSpacing(7)

        v_esq.addWidget(QLabel("<b style='color:#38BDF8; letter-spacing:0.6px;'>FILTROS ESPACIAIS</b>"))

        v_esq.addWidget(QLabel("<span style='color:#94A3B8; font-size:11px;'>Categoria / Ação:</span>"))
        self.combo_heatmap = QComboBox()
        self.combo_heatmap.addItems(["Todas as Ações"] + list(self.todas_pistas.keys()))
        self.combo_heatmap.currentTextChanged.connect(self.desenhar_heatmap_espacial)
        v_esq.addWidget(self.combo_heatmap)

        v_esq.addWidget(QLabel("<span style='color:#94A3B8; font-size:11px;'>Etiqueta:</span>"))
        self.combo_etiqueta_spatial = QComboBox()
        self.combo_etiqueta_spatial.addItems(["Todas as Etiquetas"] + list(self.lista_labels))
        self.combo_etiqueta_spatial.currentTextChanged.connect(self.desenhar_heatmap_espacial)
        v_esq.addWidget(self.combo_etiqueta_spatial)

        v_esq.addSpacing(4)
        v_esq.addWidget(QLabel("<b style='color:#38BDF8; letter-spacing:0.6px;'>CAMADAS DO RELVADO</b>"))

        self.chk_spatial_heatmap = QCheckBox("Mapa de Calor (Heatmap)")
        self.chk_spatial_heatmap.setChecked(True)
        self.chk_spatial_heatmap.toggled.connect(self.desenhar_heatmap_espacial)
        v_esq.addWidget(self.chk_spatial_heatmap)

        self.chk_spatial_pontos = QCheckBox("Pontos / Marcadores de Eventos")
        self.chk_spatial_pontos.setChecked(True)
        self.chk_spatial_pontos.toggled.connect(self.desenhar_heatmap_espacial)
        v_esq.addWidget(self.chk_spatial_pontos)

        self.chk_spatial_zonas = QCheckBox("Grelha Tática / Zonas")
        self.chk_spatial_zonas.setChecked(True)
        self.chk_spatial_zonas.toggled.connect(self.desenhar_heatmap_espacial)
        v_esq.addWidget(self.chk_spatial_zonas)

        btn_upd = QPushButton("Atualizar Diagrama")
        btn_upd.setObjectName("AccentBtn")
        btn_upd.clicked.connect(self.desenhar_heatmap_espacial)
        v_esq.addWidget(btn_upd)

        v_esq.addSpacing(4)
        v_esq.addWidget(QLabel("<b style='color:#94A3B8;'>RESUMO ZONAL</b>"))
        self.txt_resumo_zonas = QTextEdit()
        self.txt_resumo_zonas.setReadOnly(True)
        v_esq.addWidget(self.txt_resumo_zonas, 1)
        splitter.addWidget(f_esq)

        self.lbl_heatmap_canvas = QLabel()
        self.lbl_heatmap_canvas.setAlignment(Qt.AlignmentFlag.AlignCenter)
        self.lbl_heatmap_canvas.setStyleSheet("background-color: #000000; border: 1px solid #181822; border-radius: 8px;")
        splitter.addWidget(self.lbl_heatmap_canvas)
        splitter.setSizes([280, 1200])

    def escolher_ficheiro_video(self):
        caminho, _ = QFileDialog.getOpenFileName(self, "Selecionar Vídeo", "", "Vídeos (*.mp4 *.mov *.avi *.mkv *.ts)")
        if caminho:
            self.caminho_video = caminho
            ext = os.path.splitext(caminho)[1].lower()
            if ext not in (".mp4", ".mov"):
                resp = QMessageBox.question(
                    self, "Otimização Recomendada",
                    f"O ficheiro está no formato {ext.upper()}. Deseja otimizá-lo agora para MP4 (H.264) para obter máxima fluidez?"
                )
                if resp == QMessageBox.StandardButton.Yes:
                    self.otimizar_video_integrado()
                    return
            self.iniciar_video()

    def iniciar_video(self):
        if not os.path.exists(self.caminho_video):
            QMessageBox.critical(self, "Erro", f"Ficheiro não encontrado:\n{self.caminho_video}")
            return

        cap_tmp = cv2.VideoCapture(self.caminho_video)
        self.fps = cap_tmp.get(cv2.CAP_PROP_FPS) or 25.0
        cap_tmp.release()

        self.media_player.setSource(QUrl.fromLocalFile(self.caminho_video))
        if self.tabs.currentIndex() == 1:
            self.media_player.setVideoOutput(self.video_widget_studio)
        else:
            self.media_player.setVideoOutput(self.video_widget)
        self.media_player.play()
        self.btn_play.definir_modo("pause")

    def alternar_play_pause(self):
        if self.media_player.playbackState() == QMediaPlayer.PlaybackState.PlayingState:
            self.media_player.pause()
            self.btn_play.definir_modo("play")
        else:
            self.media_player.play()
            self.btn_play.definir_modo("pause")

    def ao_escolher_velocidade_combo(self, idx):
        if 0 <= idx < len(self.lista_velocidades):
            self.idx_velocidade = idx
            vel = self.lista_velocidades[idx]
            if self.tabs.currentIndex() == 0:
                self.media_player.setPlaybackRate(vel)

    def ciclar_velocidade(self):
        self.idx_velocidade = (self.idx_velocidade + 1) % len(self.lista_velocidades)
        self.combo_velocidade.setCurrentIndex(self.idx_velocidade)

    def saltar_segundos(self, delta):
        self.playlist_ativa = []
        alvo = max(0.0, min(self.duracao_total, self.tempo_atual + delta))
        self.ir_para_tempo(alvo)

    def ir_para_tempo(self, segundos):
        segundos = max(0.0, min(self.duracao_total, segundos))
        self.tempo_atual = segundos
        self.media_player.setPosition(int(segundos * 1000))
        self.lbl_tempo_atual.setText(self.formatar_tempo(segundos, True))
        self.timeline_widget.update()

    def ao_mudar_posicao_gpu(self, pos_ms):
        if self.tabs.currentIndex() == 1:
            self.ao_mudar_posicao_studio_player(pos_ms)
            return

        self.tempo_atual = pos_ms / 1000.0
        self.lbl_tempo_atual.setText(self.formatar_tempo(self.tempo_atual, True))

        if not self.arrastando_slider and self.duracao_total > 0:
            val = int((self.tempo_atual / self.duracao_total) * 1000)
            self.slider_progresso.setValue(val)

        if self.playlist_ativa:
            corte = self.playlist_ativa[self.idx_playlist]
            if self.tempo_atual >= corte["fim"]:
                self.idx_playlist += 1
                if self.idx_playlist < len(self.playlist_ativa):
                    prox = self.playlist_ativa[self.idx_playlist]
                    self.ir_para_tempo(prox["inicio"])
                else:
                    self.playlist_ativa = []
                    self.media_player.pause()
                    self.btn_play.definir_modo("play")

        self.timeline_widget.update()

    def ao_mudar_duracao_gpu(self, dur_ms):
        self.duracao_total = dur_ms / 1000.0
        self.lbl_tempo_total.setText(self.formatar_tempo(self.duracao_total, False))
        self.timeline_widget.update()

    def ao_pressionar_slider(self):
        self.arrastando_slider = True

    def ao_mover_slider(self, val):
        if self.duracao_total > 0:
            t = (val / 1000.0) * self.duracao_total
            self.tempo_atual = t
            self.lbl_tempo_atual.setText(self.formatar_tempo(t, True))
            self.timeline_widget.update()

    def ao_soltar_slider(self):
        self.arrastando_slider = False
        if self.duracao_total > 0:
            t = (self.slider_progresso.value() / 1000.0) * self.duracao_total
            self.ir_para_tempo(t)

    def alterar_zoom_timeline(self, val):
        self.janela_zoom_segundos = float(val)
        self.timeline_widget.update()

    def formatar_tempo(self, segundos, com_centesimos=True):
        segundos = max(0.0, segundos)
        mins = int(segundos // 60)
        segs = int(segundos % 60)
        if com_centesimos:
            centesimos = int((segundos - int(segundos)) * 100)
            return f"{mins:02d}:{segs:02d}.{centesimos:02d}"
        return f"{mins:02d}:{segs:02d}"

    def ao_selecionar_zona_campo(self, nome_zona, coord, corredor):
        self.zona_campo_atual = nome_zona
        self.coord_campo_atual = coord

        zonas_exclusivas = {
            "Corredor Esq", "Corredor Central", "Corredor Dir",
            "Ponta Esq", "Lateral Esq", "Central / Pivot", "Lateral Dir", "Ponta Dir",
            "Garrafão", "Ala / 3P", "Topo / Chave",
            "Zona 1", "Zona 2", "Zona 3", "Zona 4", "Zona 5", "Zona 6", "Zona 7", "Zona 8", "Zona 9"
        }
        for z in list(self.labels_ativas):
            if z in zonas_exclusivas:
                self.labels_ativas.remove(z)

        alvo = corredor if corredor in self.lista_labels else (nome_zona if nome_zona in self.lista_labels else "")
        if alvo:
            self.labels_ativas.add(alvo)

        self.freeform_canvas.atualizar_visual_todos()

    def toggle_label(self, lbl):
        if lbl in self.labels_ativas:
            self.labels_ativas.remove(lbl)
        else:
            self.labels_ativas.add(lbl)
        self.freeform_canvas.atualizar_visual_todos()

    def acionar_item_codificacao(self, item):
        if self.duracao_total <= 0:
            return
        nome = item["nome"]
        tipo = item["tipo"]
        t_agora = self.tempo_atual

        pre_tipo = item.get("pre_tipo", "clique" if tipo == "momento" else "segundos")
        pre_val = float(item.get("pre_val", 0.0 if tipo == "momento" else 10.0))
        pos_tipo = item.get("pos_tipo", "clique" if tipo == "momento" else "segundos")
        pos_val = float(item.get("pos_val", 0.0 if tipo == "momento" else 10.0))

        if tipo == "momento" and pre_tipo == "clique" and pos_tipo == "clique":
            self.alternar_momento(nome)
            return

        if nome in self.acoes_em_gravacao:
            dados_rec = self.acoes_em_gravacao.pop(nome)
            t_ini = dados_rec["inicio"]
            t_mom = dados_rec["momento"]
            t_fim = max(t_ini + 0.5, round(t_agora, 2))
            self.eventos_registados.append({
                "acao": nome, "inicio": round(t_ini, 2), "fim": t_fim,
                "momento": round(t_mom, 2), "labels": list(self.labels_ativas),
                "zona": self.zona_campo_atual, "coord": self.coord_campo_atual, "nota": ""
            })
            self.bloco_selecionado = len(self.eventos_registados) - 1
            self.guardar_autosave()
            self.freeform_canvas.atualizar_visual_todos()
            self.atualizar_dashboard()
            self.timeline_widget.update()
            return

        t_inicio_calc = t_agora if pre_tipo == "clique" else max(0.0, t_agora - pre_val)

        if pos_tipo == "clique":
            self.acoes_em_gravacao[nome] = {"inicio": t_inicio_calc, "momento": t_agora}
            if "link_momento" in item and item["link_momento"] in self.momentos_jogo:
                if self.momento_ativo != item["link_momento"]:
                    self.alternar_momento(item["link_momento"])
            self.freeform_canvas.atualizar_visual_todos()
            self.timeline_widget.update()
            return

        t_fim_calc = min(self.duracao_total, t_agora + pos_val)
        self.eventos_registados.append({
            "acao": nome, "inicio": round(t_inicio_calc, 2),
            "fim": round(max(t_inicio_calc + 0.5, t_fim_calc), 2),
            "momento": round(t_agora, 2),
            "labels": list(self.labels_ativas), "zona": self.zona_campo_atual,
            "coord": self.coord_campo_atual, "nota": ""
        })
        self.bloco_selecionado = len(self.eventos_registados) - 1

        if "link_momento" in item and item["link_momento"] in self.momentos_jogo:
            if self.momento_ativo != item["link_momento"]:
                self.alternar_momento(item["link_momento"])

        self.guardar_autosave()
        self.atualizar_dashboard()
        self.timeline_widget.update()

    def alternar_momento(self, novo_momento):
        if self.duracao_total <= 0 or novo_momento not in self.momentos_jogo:
            return
        t_agora = self.tempo_atual
        if self.momento_ativo is not None and self.momento_ativo in self.momentos_jogo:
            antigo = self.momento_ativo
            self.eventos_registados.append({
                "acao": antigo, "inicio": round(self.inicio_momento_ativo, 2),
                "fim": max(self.inicio_momento_ativo + 0.5, round(t_agora, 2)),
                "momento": round(self.inicio_momento_ativo, 2),
                "labels": list(self.labels_ativas), "zona": self.zona_campo_atual,
                "coord": self.coord_campo_atual, "nota": ""
            })
            self.guardar_autosave()
            if antigo == novo_momento:
                self.momento_ativo = None
                self.freeform_canvas.atualizar_visual_todos()
                self.atualizar_dashboard()
                self.timeline_widget.update()
                return

        self.momento_ativo = novo_momento
        self.inicio_momento_ativo = t_agora
        self.freeform_canvas.atualizar_visual_todos()
        self.atualizar_dashboard()
        self.timeline_widget.update()

    def atualizar_dashboard(self):
        self.db.sincronizar_lista(self.eventos_registados)
        contagem = self.db.contagem_por_acao()
        for acao, lbl in self.dash_labels.items():
            try:
                lbl.setText(str(contagem.get(acao, 0)))
            except RuntimeError:
                pass

    def guardar_nota_corte(self):
        if self.bloco_selecionado is not None and 0 <= self.bloco_selecionado < len(self.eventos_registados):
            self.eventos_registados[self.bloco_selecionado]["nota"] = self.entry_nota.text().strip()
            self.db.sincronizar_lista(self.eventos_registados)
            self.guardar_autosave()
            self.timeline_widget.update()

    def iniciar_playlist_cortes(self, lista, dialog=None):
        if dialog:
            dialog.accept()
        if not lista:
            return
        self.playlist_ativa = lista
        self.idx_playlist = 0
        self.ir_para_tempo(lista[0]["inicio"])
        self.media_player.play()
        self.btn_play.definir_modo("pause")

    def abrir_janela_matrix(self):
        dlg = QDialog(self)
        dlg.setWindowTitle("Matriz de Cruzamento de Dados")
        dlg.resize(780, 440)
        lay = QVBoxLayout(dlg)
        grid = QGridLayout()

        colunas = ["Total"] + self.lista_labels
        grid.addWidget(QLabel("<b>Categoria</b>"), 0, 0)
        for c_idx, col in enumerate(colunas):
            grid.addWidget(QLabel(f"<b>{col}</b>"), 0, c_idx + 1)

        for r_idx, acao in enumerate(self.todas_pistas.keys()):
            grid.addWidget(QLabel(acao), r_idx + 1, 0)
            for c_idx, col in enumerate(colunas):
                cortes = [ev for ev in self.eventos_registados if ev["acao"] == acao and (col == "Total" or col in ev["labels"])]
                qtd = len(cortes)
                btn = QPushButton(str(qtd))
                btn.setEnabled(qtd > 0)
                if qtd > 0:
                    btn.setObjectName("AccentBtn")
                btn.clicked.connect(lambda checked, l=cortes, d=dlg: self.iniciar_playlist_cortes(l, d))
                grid.addWidget(btn, r_idx + 1, c_idx + 1)

        lay.addLayout(grid)
        dlg.exec()

    def importar_todos_para_estudio(self):
        if not self.eventos_registados:
            QMessageBox.warning(self, "Aviso", "Registe primeiro cortes no Match Desk.")
            return
        self.importar_lista_especifica_para_estudio(self.eventos_registados)

    def importar_lista_especifica_para_estudio(self, lista_evs):
        self.studio_clips = []
        for ev in lista_evs:
            self.studio_clips.append({
                "tipo_clip": "video",
                "acao": ev["acao"], "inicio": ev["inicio"], "fim": ev["fim"],
                "momento": ev["momento"], "labels": list(ev["labels"]),
                "nota": ev["nota"], "zona": ev.get("zona", ""),
                "keyframes": []
            })
        self.tabs.setCurrentIndex(1)
        self.atualizar_lista_ui_estudio()
        if self.studio_clips:
            self.lista_clips_studio_w.setCurrentRow(0)

    def atualizar_lista_ui_estudio(self):
        self.lista_clips_studio_w.blockSignals(True)
        self.lista_clips_studio_w.clear()
        for idx, clip in enumerate(self.studio_clips):
            t_i = self.formatar_tempo(clip["inicio"], False)
            t_f = self.formatar_tempo(clip["fim"], False)
            n_kf = len(clip.get("keyframes", []))
            badge_kf = f"  •  [{n_kf} Pausa(s)]" if n_kf > 0 else ""
            self.lista_clips_studio_w.addItem(f"{idx+1}. {clip['acao']} ({t_i} – {t_f}){badge_kf}")
        if self.idx_clip_studio is not None and 0 <= self.idx_clip_studio < len(self.studio_clips):
            self.lista_clips_studio_w.setCurrentRow(self.idx_clip_studio)
        self.lista_clips_studio_w.blockSignals(False)

    def atualizar_lista_keyframes_ui(self):
        self.lista_keyframes_w.blockSignals(True)
        self.lista_keyframes_w.clear()
        if self.idx_clip_studio is not None and 0 <= self.idx_clip_studio < len(self.studio_clips):
            clip = self.studio_clips[self.idx_clip_studio]
            for k_idx, kf in enumerate(clip.get("keyframes", [])):
                t_str = self.formatar_tempo(kf["tempo"], True)
                dur_s = kf.get("duracao", 3.0)
                n_fx = len(kf.get("efeitos", []))
                self.lista_keyframes_w.addItem(f"P{k_idx+1}  |  Aos {t_str}  ({dur_s}s • {n_fx} Efeitos)")
            if self.idx_keyframe_studio is not None and 0 <= self.idx_keyframe_studio < len(clip.get("keyframes", [])):
                self.lista_keyframes_w.setCurrentRow(self.idx_keyframe_studio)
        self.lista_keyframes_w.blockSignals(False)

    def preparar_cache_skim_clip(self, clip):
        self.studio_skim_cache = []
        if not os.path.exists(self.caminho_video):
            return
        if hasattr(self, "worker_skim") and self.worker_skim is not None and self.worker_skim.isRunning():
            self.worker_skim.requestInterruption()
            self.worker_skim.wait(50)

        class _SkimWorker(QThread):
            pronto = pyqtSignal(list)

            def __init__(self, path, inicio, fim, fps):
                super().__init__()
                self.path, self.inicio, self.fim, self.fps = path, inicio, fim, fps

            def run(self):
                cache_local = []
                dur = max(0.5, self.fim - self.inicio)
                n_frames = min(35, max(12, int(dur * 2.2)))
                cap = cv2.VideoCapture(self.path)
                for i in range(n_frames):
                    if self.isInterruptionRequested():
                        break
                    t_f = self.inicio + (i / max(1, n_frames - 1)) * dur
                    cap.set(cv2.CAP_PROP_POS_FRAMES, int(t_f * self.fps))
                    ret, frame = cap.read()
                    if ret:
                        fh, fw = frame.shape[:2]
                        escala = min(1.0, 854.0 / max(1, fw))
                        mini = cv2.resize(frame, (int(fw * escala), int(fh * escala)))
                        rgb = cv2.cvtColor(mini, cv2.COLOR_BGR2RGB)
                        qimg = QImage(rgb.data, rgb.shape[1], rgb.shape[0], 3 * rgb.shape[1], QImage.Format.Format_RGB888).copy()
                        cache_local.append((t_f, qimg))
                cap.release()
                self.pronto.emit(cache_local)

        self.worker_skim = _SkimWorker(self.caminho_video, clip["inicio"], clip["fim"], self.fps)
        self.worker_skim.pronto.connect(lambda lista: setattr(self, "studio_skim_cache", [(t, QPixmap.fromImage(qi)) for t, qi in lista]))
        self.worker_skim.start()

    def mostrar_frame_skim_instantaneo(self, tempo_alvo):
        cache = getattr(self, "studio_skim_cache", [])
        if not cache:
            return
        _, pix_orig = min(cache, key=lambda item: abs(item[0] - tempo_alvo))
        dpr = self.devicePixelRatioF()
        pix = pix_orig.scaled(
            int(max(400, self.canvas_studio.width()) * dpr),
            int(max(300, self.canvas_studio.height()) * dpr),
            Qt.AspectRatioMode.KeepAspectRatio,
            Qt.TransformationMode.FastTransformation
        )
        pix.setDevicePixelRatio(dpr)
        self.stack_visor_studio.setCurrentIndex(1)
        self.canvas_studio.setPixmap(pix)

    def selecionar_clip_estudio(self, idx):
        if not (0 <= idx < len(self.studio_clips)):
            return
        self.timer_pausa_studio.stop()
        self.idx_clip_studio = idx
        self.idx_keyframe_studio = None
        self.canvas_studio.efeito_rascunho = None
        self.barra_teste_efeito.setVisible(False)
        self.studio_keyframes_ja_pausados.clear()
        self._ultimo_t_studio = self.studio_clips[idx]["inicio"] - 0.1
        self.preparar_cache_skim_clip(self.studio_clips[idx])
        self.atualizar_lista_keyframes_ui()
        self.ativar_modo_video_intero_studio(reiniciar_do_inicio=True)

    def ativar_modo_video_intero_studio(self, reiniciar_do_inicio=False):
        if self.idx_clip_studio is None or not (0 <= self.idx_clip_studio < len(self.studio_clips)):
            return
        self.timer_pausa_studio.stop()
        self.canvas_studio.efeito_rascunho = None
        self.barra_teste_efeito.setVisible(False)
        clip = self.studio_clips[self.idx_clip_studio]
        self.stack_visor_studio.setCurrentIndex(0)
        self.btn_voltar_video_corte.setVisible(False)
        self.lbl_estado_visor_studio.setText(
            f"<b style='color:#38BDF8; font-size:13px;'>MODO VÍDEO CONTÍNUO — {clip['acao'].upper()}</b> "
            f"<span style='color:#94A3B8;'>(Passe o rato na timeline ou clique em '+ Adicionar Pausa Tática')</span>"
        )
        if reiniciar_do_inicio or self.studio_tempo_atual < clip["inicio"] or self.studio_tempo_atual >= clip["fim"]:
            self.studio_keyframes_ja_pausados.clear()
            self.studio_tempo_atual = clip["inicio"]
            self._ultimo_t_studio = clip["inicio"] - 0.1
            self.media_player.setPosition(int(clip["inicio"] * 1000))
        self.media_player.play()
        self.btn_play_studio.definir_modo("pause")
        if hasattr(self, "studio_nle_timeline"):
            self.studio_nle_timeline.update()

    def alternar_play_pause_studio(self):
        if self.idx_clip_studio is None:
            return
        self.timer_pausa_studio.stop()
        if self.stack_visor_studio.currentIndex() == 1:
            self.ativar_modo_video_intero_studio(reiniciar_do_inicio=False)
            return

        if self.media_player.playbackState() == QMediaPlayer.PlaybackState.PlayingState:
            self.media_player.pause()
            self.btn_play_studio.definir_modo("play")
        else:
            clip = self.studio_clips[self.idx_clip_studio]
            if self.studio_tempo_atual >= clip["fim"] - 0.15:
                self.studio_keyframes_ja_pausados.clear()
                self.studio_tempo_atual = clip["inicio"]
                self._ultimo_t_studio = clip["inicio"] - 0.1
                self.media_player.setPosition(int(clip["inicio"] * 1000))
            self.media_player.play()
            self.btn_play_studio.definir_modo("pause")

    def ao_mudar_posicao_studio_player(self, pos_ms):
        if self.idx_clip_studio is None or not (0 <= self.idx_clip_studio < len(self.studio_clips)):
            return
        if self.media_player.playbackState() == QMediaPlayer.PlaybackState.PlayingState:
            t_ant = getattr(self, "_ultimo_t_studio", self.studio_tempo_atual)
            self.studio_tempo_atual = pos_ms / 1000.0
            clip = self.studio_clips[self.idx_clip_studio]

            if hasattr(self, "studio_nle_timeline"):
                self.studio_nle_timeline.update()
            self.lbl_tempo_corte_studio.setText(f"{self.formatar_tempo(self.studio_tempo_atual, True)} / {self.formatar_tempo(clip['fim'], True)}")

            if self.stack_visor_studio.currentIndex() == 0:
                for k_idx, kf in enumerate(clip.get("keyframes", [])):
                    if k_idx not in self.studio_keyframes_ja_pausados:
                        cruzou = (t_ant <= kf["tempo"] <= self.studio_tempo_atual)
                        proximo = (0.0 <= (self.studio_tempo_atual - kf["tempo"]) <= 0.45)
                        if cruzou or proximo:
                            self.studio_keyframes_ja_pausados.add(k_idx)
                            self.media_player.pause()
                            self.btn_play_studio.definir_modo("play")
                            self._ultimo_t_studio = self.studio_tempo_atual
                            self.abrir_keyframe_no_editor(k_idx, auto_retomar=True)
                            return

            self._ultimo_t_studio = self.studio_tempo_atual

            if self.studio_tempo_atual >= clip["fim"]:
                prox_idx = self.idx_clip_studio + 1
                if prox_idx < len(self.studio_clips):
                    self.lista_clips_studio_w.setCurrentRow(prox_idx)
                else:
                    self.media_player.pause()
                    self.btn_play_studio.definir_modo("play")

    def retomar_video_studio_apos_pausa(self):
        if self.tabs.currentIndex() == 1 and self.idx_clip_studio is not None:
            self.ativar_modo_video_intero_studio(reiniciar_do_inicio=False)

    def capturar_fotograma_livetag_studio(self):
        if self.idx_clip_studio is None or not os.path.exists(self.caminho_video):
            return
        self.timer_pausa_studio.stop()
        self.media_player.pause()
        self.btn_play_studio.definir_modo("play")

        clip = self.studio_clips[self.idx_clip_studio]
        t_captura = max(clip["inicio"], min(clip["fim"], self.studio_tempo_atual or clip["momento"]))

        novo_kf = {
            "tempo": round(t_captura, 2),
            "duracao": float(self.spin_duracao_kf.value()),
            "nota": "",
            "efeitos": []
        }
        clip.setdefault("keyframes", []).append(novo_kf)
        clip["keyframes"].sort(key=lambda k: k["tempo"])
        novo_idx = clip["keyframes"].index(novo_kf)

        self.atualizar_lista_ui_estudio()
        self.atualizar_lista_keyframes_ui()
        self.abrir_keyframe_no_editor(novo_idx, auto_retomar=False)

    def ao_selecionar_keyframe_lista(self, k_idx):
        if k_idx >= 0:
            self.timer_pausa_studio.stop()
            self.media_player.pause()
            self.btn_play_studio.definir_modo("play")
            self.abrir_keyframe_no_editor(k_idx, auto_retomar=False)

    def abrir_keyframe_no_editor(self, k_idx, auto_retomar=False):
        if self.idx_clip_studio is None:
            return
        clip = self.studio_clips[self.idx_clip_studio]
        kfs = clip.get("keyframes", [])
        if not (0 <= k_idx < len(kfs)):
            return

        self.idx_keyframe_studio = k_idx
        kf = kfs[k_idx]

        self.studio_tempo_atual = kf["tempo"]
        self.media_player.setPosition(int(kf["tempo"] * 1000))
        self.lbl_tempo_corte_studio.setText(f"{self.formatar_tempo(kf['tempo'], True)} / {self.formatar_tempo(clip['fim'], True)}")

        self.spin_duracao_kf.blockSignals(True)
        self.spin_duracao_kf.setValue(float(kf.get("duracao", 3.0)))
        self.spin_duracao_kf.blockSignals(False)

        self.edt_nota_studio.blockSignals(True)
        self.edt_nota_studio.setText(kf.get("nota", ""))
        self.edt_nota_studio.blockSignals(False)

        cap = cv2.VideoCapture(self.caminho_video)
        cap.set(cv2.CAP_PROP_POS_FRAMES, int(kf["tempo"] * self.fps))
        ret, frame = cap.read()
        cap.release()
        if ret:
            self.frame_studio_original = frame
            self.masc_relva_cache = self.calcular_mascara_relva(frame)

            self.atualizar_preview_estudio()
            self.stack_visor_studio.setCurrentIndex(1)
            self.btn_voltar_video_corte.setVisible(not auto_retomar)
            self.lbl_estado_visor_studio.setText(
                f"<b style='color:#F59E0B; font-size:13px;'>PAUSA TÁTICA P{k_idx+1} (Aos {self.formatar_tempo(kf['tempo'], True)} • {kf['duracao']}s)</b> "
                f"<span style='color:#94A3B8;'>— Clique direito num efeito à direita para predefinir a cor</span>"
            )
            if auto_retomar:
                self.timer_pausa_studio.start(int(float(kf.get("duracao", 3.0)) * 1000))

    def iniciar_calibracao_homografia(self):
        if self.frame_studio_original is None:
            QMessageBox.information(self, "Calibração", "Abra primeiro uma Pausa Tática para calibrar o campo.")
            return
        self.modo_calibracao = True
        self.pontos_calibracao = []
        self.lbl_estado_visor_studio.setText(
            "<b style='color:#22C55E; font-size:13px;'>CALIBRAÇÃO 4 PONTOS:</b> "
            "Clique nos 4 cantos da área de referência (Sup. Esq → Sup. Dir → Inf. Dir → Inf. Esq)"
        )
        self.atualizar_preview_estudio()

    def finalizar_calibracao_homografia(self):
        h, w = self.frame_studio_original.shape[:2]
        pts_px = np.array([[nx * w, ny * h] for nx, ny in self.pontos_calibracao], dtype=np.float32)
        pts_metros = np.array([[0.0, 0.0], [40.32, 0.0], [40.32, 16.5], [0.0, 16.5]], dtype=np.float32)
        self.matriz_homografia = cv2.getPerspectiveTransform(pts_px, pts_metros)
        self.modo_calibracao = False
        self.pontos_calibracao = []
        self.lbl_estado_visor_studio.setText("<b style='color:#22C55E; font-size:13px;'>CAMPO CALIBRADO (HOMOGRAFIA ATIVA) — Distâncias reais em metros</b>")
        self.atualizar_preview_estudio()

    def previsualizar_keyframe_temporario(self, k_idx):
        if self.idx_clip_studio is None:
            return
        clip = self.studio_clips[self.idx_clip_studio]
        kfs = clip.get("keyframes", [])
        if not (0 <= k_idx < len(kfs)):
            return
        kf = kfs[k_idx]
        cap = cv2.VideoCapture(self.caminho_video)
        cap.set(cv2.CAP_PROP_POS_FRAMES, int(kf["tempo"] * self.fps))
        ret, frame = cap.read()
        cap.release()
        if ret:
            img_bgr = self.desenhar_efeitos_broadcast(
                frame, kf.get("efeitos", []),
                self.chk_minimapa.isChecked(),
                self.chk_mask_3d.isChecked(),
                legenda_inferior=kf.get("nota", ""),
                matriz_homografia=self.matriz_homografia
            )
            img_rgb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB)
            h, w, ch = img_rgb.shape
            qimg = QImage(img_rgb.data, w, h, ch * w, QImage.Format.Format_RGB888)
            dpr = self.devicePixelRatioF()
            pix = QPixmap.fromImage(qimg).scaled(
                int(max(400, self.canvas_studio.width()) * dpr),
                int(max(300, self.canvas_studio.height()) * dpr),
                Qt.AspectRatioMode.KeepAspectRatio,
                Qt.TransformationMode.FastTransformation
            )
            pix.setDevicePixelRatio(dpr)
            self.stack_visor_studio.setCurrentIndex(1)
            self.canvas_studio.setPixmap(pix)

    def obter_keyframe_studio_ativo(self):
        if self.idx_clip_studio is None:
            return None
        clip = self.studio_clips[self.idx_clip_studio]
        kfs = clip.setdefault("keyframes", [])
        if self.idx_keyframe_studio is None or not (0 <= self.idx_keyframe_studio < len(kfs)):
            self.capturar_fotograma_livetag_studio()
        return kfs[self.idx_keyframe_studio] if (self.idx_keyframe_studio is not None and 0 <= self.idx_keyframe_studio < len(kfs)) else None

    def ao_mudar_duracao_keyframe(self, val):
        kf = self.obter_keyframe_studio_ativo()
        if kf is not None:
            kf["duracao"] = float(val)
            self.atualizar_lista_keyframes_ui()
            if hasattr(self, "studio_nle_timeline"):
                self.studio_nle_timeline.update()

    def ao_editar_nota_studio(self, txt):
        kf = self.obter_keyframe_studio_ativo()
        if kf is not None:
            kf["nota"] = txt.strip()
            self.atualizar_preview_estudio()

    def eliminar_keyframe_ativo(self):
        if self.idx_clip_studio is None or self.idx_keyframe_studio is None:
            return
        clip = self.studio_clips[self.idx_clip_studio]
        kfs = clip.get("keyframes", [])
        if 0 <= self.idx_keyframe_studio < len(kfs):
            kfs.pop(self.idx_keyframe_studio)
            self.idx_keyframe_studio = None
            self.canvas_studio.efeito_rascunho = None
            self.barra_teste_efeito.setVisible(False)
            self.atualizar_lista_ui_estudio()
            self.atualizar_lista_keyframes_ui()
            self.ativar_modo_video_intero_studio(reiniciar_do_inicio=False)

    def mover_clip_estudio(self, direcao):
        if self.idx_clip_studio is None:
            return
        novo_idx = self.idx_clip_studio + direcao
        if 0 <= novo_idx < len(self.studio_clips):
            self.studio_clips[self.idx_clip_studio], self.studio_clips[novo_idx] = (
                self.studio_clips[novo_idx], self.studio_clips[self.idx_clip_studio]
            )
            self.idx_clip_studio = novo_idx
            self.atualizar_lista_ui_estudio()

    def remover_clip_estudio(self):
        if self.idx_clip_studio is not None and 0 <= self.idx_clip_studio < len(self.studio_clips):
            self.studio_clips.pop(self.idx_clip_studio)
            if not self.studio_clips:
                self.idx_clip_studio = None
                self.canvas_studio.clear()
                self.atualizar_lista_ui_estudio()
                self.atualizar_lista_keyframes_ui()
            else:
                novo = min(self.idx_clip_studio, len(self.studio_clips) - 1)
                self.atualizar_lista_ui_estudio()
                self.lista_clips_studio_w.setCurrentRow(novo)

    def selecionar_ferramenta_studio(self, cod):
        self.ferramenta_studio = cod
        for k, b in self.botoes_ferramentas.items():
            if k == cod:
                b.setStyleSheet("background-color: #0284C7; color: #FFFFFF; border: 1px solid #38BDF8; text-align: left; padding: 6px 10px;")
            else:
                b.setStyleSheet("text-align: left; padding: 6px 10px;")

    def criar_icone_efeito(self, cod, cor_hex):
        pix = QPixmap(20, 20)
        pix.fill(Qt.GlobalColor.transparent)
        p = QPainter(pix)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        c = QColor(cor_hex)
        p.setPen(QPen(c, 2.0))
        if cod == "anel":
            p.drawEllipse(QRectF(2, 6, 16, 8))
        elif cod == "spotlight":
            p.setBrush(QBrush(QColor(c.red(), c.green(), c.blue(), 90)))
            p.drawPolygon(QPolygonF([QPointF(10, 2), QPointF(3, 17), QPointF(17, 17)]))
        elif cod == "linha":
            p.setBrush(QBrush(QColor(c.red(), c.green(), c.blue(), 80)))
            p.drawPolygon(QPolygonF([QPointF(3, 14), QPointF(6, 4), QPointF(16, 6), QPointF(14, 16)]))
        elif cod == "seta":
            p.drawArc(QRectF(3, 5, 14, 12), 0, 180 * 16)
            p.drawLine(14, 7, 17, 11)
        elif cod in ("cone", "sombra"):
            p.setBrush(QBrush(QColor(c.red(), c.green(), c.blue(), 110)))
            p.drawPie(QRectF(2, 2, 16, 16), 30 * 16, 120 * 16)
        elif cod == "ghost":
            p.drawEllipse(QRectF(3, 4, 6, 6))
            p.setPen(QPen(c, 1.8, Qt.PenStyle.DashLine))
            p.drawEllipse(QRectF(11, 10, 6, 6))
        elif cod == "zoom":
            p.drawEllipse(QRectF(3, 3, 10, 10))
            p.drawLine(12, 12, 17, 17)
        elif cod == "livre":
            p.drawLine(3, 16, 8, 6)
            p.drawLine(8, 6, 13, 13)
            p.drawLine(13, 13, 17, 4)
        elif cod == "texto":
            p.setFont(QFont("Helvetica Neue", 12, QFont.Weight.Bold))
            p.drawText(QRectF(0, 0, 20, 20), Qt.AlignmentFlag.AlignCenter, "T")
        elif cod == "laser":
            p.setBrush(QBrush(QColor("#EF4444")))
            p.setPen(Qt.PenStyle.NoPen)
            p.drawEllipse(QPointF(10, 10), 5, 5)
        p.end()
        return QIcon(pix)

    def predefinir_cor_ferramenta(self, cod):
        c = QColorDialog.getColor(QColor(self.cores_efeitos.get(cod, "#00E5FF")), self, "Cor Padrão do Efeito")
        if c.isValid():
            self.cores_efeitos[cod] = c.name().upper()
            if cod in self.botoes_ferramentas:
                self.botoes_ferramentas[cod].setIcon(self.criar_icone_efeito(cod, self.cores_efeitos[cod]))

    def escolher_cor_efeito_rascunho(self):
        ef = self.canvas_studio.efeito_rascunho
        if ef is not None:
            c = QColorDialog.getColor(QColor(ef.get("cor", "#00E5FF")), self, "Cor do Efeito")
            if c.isValid():
                cor_u = c.name().upper()
                ef["cor"] = cor_u
                self.cores_efeitos[ef["tipo"]] = cor_u
                if ef["tipo"] in self.botoes_ferramentas:
                    self.botoes_ferramentas[ef["tipo"]].setIcon(self.criar_icone_efeito(ef["tipo"], cor_u))
                self.atualizar_preview_estudio(efeito_preview=ef)

    def desfazer_efeito_studio(self):
        if self.canvas_studio.efeito_rascunho is not None:
            self.descartar_efeito_rascunho()
            return
        kf = self.obter_keyframe_studio_ativo()
        if kf is not None and kf.get("efeitos"):
            kf["efeitos"].pop()
            self.canvas_studio.efeito_selecionado_idx = None
            self.atualizar_lista_keyframes_ui()
            self.atualizar_preview_estudio()

    def limpar_efeitos_studio(self):
        self.canvas_studio.efeito_rascunho = None
        self.barra_teste_efeito.setVisible(False)
        kf = self.obter_keyframe_studio_ativo()
        if kf is not None:
            kf["efeitos"].clear()
            self.canvas_studio.efeito_selecionado_idx = None
            self.atualizar_lista_keyframes_ui()
            self.atualizar_preview_estudio()

    def confirmar_efeito_rascunho(self):
        kf = self.obter_keyframe_studio_ativo()
        if kf is not None and self.canvas_studio.efeito_rascunho is not None:
            kf.setdefault("efeitos", []).append(self.canvas_studio.efeito_rascunho)
            self.canvas_studio.efeito_rascunho = None
            self.barra_teste_efeito.setVisible(False)
            self.studio_keyframes_ja_pausados.clear()
            self.atualizar_lista_ui_estudio()
            self.atualizar_lista_keyframes_ui()
            self.atualizar_preview_estudio()

    def descartar_efeito_rascunho(self):
        self.canvas_studio.efeito_rascunho = None
        self.barra_teste_efeito.setVisible(False)
        self.atualizar_preview_estudio()

    def auto_detetar_jogadores(self):
        kf = self.obter_keyframe_studio_ativo()
        if kf is None or self.frame_studio_original is None:
            return
        frame = self.frame_studio_original.copy()
        h, w = frame.shape[:2]
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
        relva = cv2.inRange(hsv, np.array([28, 35, 35]), np.array([85, 255, 255]))
        zona = cv2.morphologyEx(relva, cv2.MORPH_CLOSE, np.ones((15, 15), np.uint8))
        sil = cv2.dilate(cv2.bitwise_and(cv2.bitwise_not(relva), zona), np.ones((5, 5), np.uint8), iterations=2)
        cnts, _ = cv2.findContours(sil, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        det = []
        for c in cnts:
            x, y, bw, bh = cv2.boundingRect(c)
            if (h * 0.035) < bh < (h * 0.22) and (w * 0.008) < bw < (w * 0.08) and (bh / max(1, bw)) > 1.15 and y > h * 0.12:
                det.append(((x + bw / 2) / w, (y + bh) / h, bw * bh))
        det.sort(key=lambda i: i[2], reverse=True)
        cor_anel = self.cores_efeitos.get("anel", "#00E5FF")
        for nx, ny, _ in det[:10]:
            kf["efeitos"].append({"tipo": "anel", "cor": cor_anel, "nx": nx, "ny": ny})
        self.atualizar_lista_keyframes_ui()
        self.atualizar_preview_estudio()

    @staticmethod
    def calcular_mascara_relva(frame_bgr):
        b, g, r = cv2.split(frame_bgr.astype(np.int16))
        exg = (2 * g) - r - b
        masc_exg = np.where(exg > 18, 255, 0).astype(np.uint8)

        hsv = cv2.cvtColor(frame_bgr, cv2.COLOR_BGR2HSV)
        masc_hsv = cv2.inRange(hsv, np.array([25, 25, 25]), np.array([95, 255, 245]))

        masc = cv2.bitwise_and(masc_exg, masc_hsv)
        kernel = np.ones((3, 3), np.uint8)
        masc = cv2.morphologyEx(masc, cv2.MORPH_OPEN, kernel)
        masc = cv2.GaussianBlur(masc, (5, 5), 0)
        return (masc / 255.0)[:, :, np.newaxis]

    @staticmethod
    def desenhar_efeitos_broadcast(frame, efeitos, mostrar_minimapa=True, mascarar_relva_3d=True, legenda_inferior="", laser_coord=None, masc_relva_precalc=None, matriz_homografia=None, pts_calibracao=None):
        orig = frame.copy()
        img = frame.copy()
        h, w = img.shape[:2]

        rgb_chao = np.ascontiguousarray(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
        qimg_chao = QImage(rgb_chao.data, w, h, 3 * w, QImage.Format.Format_RGB888).copy()
        pc = QPainter(qimg_chao)
        pc.setRenderHint(QPainter.RenderHint.Antialiasing, True)

        for ef in efeitos:
            tp = ef.get("tipo")
            cor_hex = ef.get("cor", "#00E5FF")
            if tp in ("sombra", "cone"):
                x1, y1 = ef["nx1"] * w, ef["ny1"] * h
                x2, y2 = ef["nx2"] * w, ef["ny2"] * h
                ang = math.atan2(y2 - y1, x2 - x1)
                dist = math.hypot(x2 - x1, y2 - y1)
                ab = 0.36 if tp == "sombra" else 0.42
                poly = QPolygonF([
                    QPointF(x1, y1),
                    QPointF(x1 + dist * math.cos(ang - ab), y1 + dist * math.sin(ang - ab)),
                    QPointF(x1 + dist * math.cos(ang + ab), y1 + dist * math.sin(ang + ab))
                ])
                c_fill = QColor(cor_hex)
                c_fill.setAlpha(155 if tp == "sombra" else 95)
                pc.setBrush(QBrush(c_fill))
                pc.setPen(QPen(QColor(cor_hex), 2.5))
                pc.drawPolygon(poly)

            elif tp == "linha":
                pts_norm = ef.get("pontos", [(ef["nx"], ef["ny"])] if "nx" in ef else [])
                if len(pts_norm) >= 2:
                    c_fill = QColor(cor_hex)
                    c_fill.setAlpha(int(ef.get("opacidade", 36) * 2.55))
                    poly = QPolygonF([QPointF(px * w, py * h) for px, py in pts_norm])
                    if len(pts_norm) >= 3:
                        pc.setBrush(QBrush(c_fill))
                        pc.setPen(QPen(QColor(cor_hex), 3.0))
                        pc.drawPolygon(poly)
                    else:
                        pc.setPen(QPen(QColor(cor_hex), 3.5))
                        pc.drawPolyline(poly)

            elif tp == "anel":
                cx, cy = ef["nx"] * w, ef["ny"] * h
                rx, ry = w * 0.028, h * 0.013
                c_halo = QColor(cor_hex)
                c_halo.setAlpha(70)
                pc.setBrush(QBrush(c_halo))
                pc.setPen(Qt.PenStyle.NoPen)
                pc.drawEllipse(QPointF(cx, cy), rx + 7, ry + 4)
                pc.setBrush(Qt.BrushStyle.NoBrush)
                pc.setPen(QPen(QColor(0, 0, 0, 180), 4.0))
                pc.drawEllipse(QPointF(cx, cy + 2), rx + 2, ry + 1)
                pc.setPen(QPen(QColor(cor_hex), 3.2))
                pc.drawEllipse(QPointF(cx, cy), rx + 2, ry + 1)
                pc.setPen(QPen(QColor("#FFFFFF"), 1.6))
                pc.drawEllipse(QPointF(cx, cy), rx - 2, max(2.0, ry - 2))

        pc.end()
        ptr_c = qimg_chao.constBits()
        ptr_c.setsize(h * w * 3)
        camada_chao = cv2.cvtColor(np.frombuffer(ptr_c, np.uint8).reshape((h, w, 3)).copy(), cv2.COLOR_RGB2BGR)

        if mascarar_relva_3d:
            if masc_relva_precalc is not None and masc_relva_precalc.shape[:2] == (h, w):
                alpha_r = masc_relva_precalc
            else:
                alpha_r = KronosProQtApp.calcular_mascara_relva(orig)
            img = (camada_chao * alpha_r + orig * (1.0 - alpha_r)).astype(np.uint8)
        else:
            img = camada_chao

        spots = [e for e in efeitos if e["tipo"] == "spotlight"]
        if spots:
            escuro = (img * 0.28).astype(np.uint8)
            masc = np.zeros((h, w), dtype=np.uint8)
            r = int(min(w, h) * 0.085)
            for s in spots:
                sx, sy = int(s["nx"] * w), int(s["ny"] * h)
                cv2.circle(masc, (sx, sy), r, 255, -1)
                beam = np.array([[sx - int(r * 0.4), 0], [sx + int(r * 0.4), 0], [sx + r, sy], [sx - r, sy]], np.int32)
                cv2.fillPoly(masc, [beam], 160)
            alpha_s = (cv2.GaussianBlur(masc, (61, 61), 0) / 255.0)[:, :, np.newaxis]
            img = (img * alpha_s + escuro * (1.0 - alpha_s)).astype(np.uint8)

        for gh in [e for e in efeitos if e["tipo"] == "ghost"]:
            x1, y1 = int(gh["nx1"] * w), int(gh["ny1"] * h)
            x2, y2 = int(gh["nx2"] * w), int(gh["ny2"] * h)
            pw, ph = int(w * 0.026), int(h * 0.065)
            sy1, sy2 = max(0, y1 - ph), min(h, y1 + ph)
            sx1, sx2 = max(0, x1 - pw), min(w, x1 + pw)
            rec_atleta = orig[sy1:sy2, sx1:sx2]
            rh_a, rw_a = rec_atleta.shape[:2]
            if rh_a > 10 and rw_a > 10:
                img[sy1:sy2, sx1:sx2] = (img[sy1:sy2, sx1:sx2] * 0.48).astype(np.uint8)
                dy1 = max(0, min(h - rh_a, y2 - rh_a // 2))
                dx1 = max(0, min(w - rw_a, x2 - rw_a // 2))
                roi_dest = img[dy1:dy1 + rh_a, dx1:dx1 + rw_a]
                masc_elip = np.zeros((rh_a, rw_a), dtype=np.uint8)
                cv2.ellipse(masc_elip, (rw_a // 2, rh_a // 2), (int(rw_a * 0.46), int(rh_a * 0.48)), 0, 0, 360, 255, -1)
                alpha_g = (cv2.GaussianBlur(masc_elip, (15, 15), 0) / 255.0)[:, :, np.newaxis]
                img[dy1:dy1 + rh_a, dx1:dx1 + rw_a] = (rec_atleta * alpha_g + roi_dest * (1.0 - alpha_g)).astype(np.uint8)

        for z in [e for e in efeitos if e["tipo"] == "zoom"]:
            nx1 = z.get("nx1", z.get("nx", 0.5))
            ny1 = z.get("ny1", z.get("ny", 0.5))
            nx2 = z.get("nx2", min(0.88, nx1 + 0.14))
            ny2 = z.get("ny2", max(0.14, ny1 - 0.16))
            cx, cy = int(nx1 * w), int(ny1 * h)
            dx_c, dy_c = int(nx2 * w), int(ny2 * h)
            rw, rh = int(w * 0.055), int(h * 0.055)
            rec = orig[max(0, cy - rh):min(h, cy + rh), max(0, cx - rw):min(w, cx + rw)]
            if rec.size > 0:
                diam = int(min(rw, rh) * 4.0)
                amp = cv2.resize(rec, (diam, diam), interpolation=cv2.INTER_CUBIC)
                dx = max(10, min(w - diam - 10, dx_c - diam // 2))
                dy = max(10, min(h - diam - 10, dy_c - diam // 2))
                roi_z = img[dy:dy + diam, dx:dx + diam]
                masc_c = np.zeros((diam, diam), dtype=np.uint8)
                cv2.circle(masc_c, (diam // 2, diam // 2), diam // 2 - 2, 255, -1, cv2.LINE_AA)
                alpha_z = (masc_c / 255.0)[:, :, np.newaxis]
                img[dy:dy + diam, dx:dx + diam] = (amp * alpha_z + roi_z * (1.0 - alpha_z)).astype(np.uint8)
                cor_z = QColor(z.get("cor", "#00E5FF"))
                bgr_z = (cor_z.blue(), cor_z.green(), cor_z.red())
                cv2.line(img, (cx, cy), (dx + diam // 2, dy + diam // 2), bgr_z, 2, cv2.LINE_AA)
                cv2.circle(img, (cx, cy), 6, bgr_z, 2, cv2.LINE_AA)
                cv2.circle(img, (dx + diam // 2, dy + diam // 2), diam // 2 - 2, bgr_z, 4, cv2.LINE_AA)
                cv2.circle(img, (dx + diam // 2, dy + diam // 2), diam // 2 - 2, (255, 255, 255), 1, cv2.LINE_AA)

        rgb_top = np.ascontiguousarray(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
        qimg = QImage(rgb_top.data, w, h, 3 * w, QImage.Format.Format_RGB888).copy()
        p = QPainter(qimg)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        p.setRenderHint(QPainter.RenderHint.TextAntialiasing, True)

        for ef in efeitos:
            tp = ef.get("tipo")
            cor_hex = ef.get("cor", "#FFD600")
            if tp == "linha":
                pts_norm = ef.get("pontos", [(ef["nx"], ef["ny"])] if "nx" in ef else [])
                for i, (px, py) in enumerate(pts_norm):
                    p.setBrush(QBrush(QColor("#FFFFFF")))
                    p.setPen(QPen(QColor(cor_hex), 2.5))
                    p.drawEllipse(QPointF(px * w, py * h), 5.0, 5.0)
                    if i < len(pts_norm) - 1:
                        p1x, p1y = px * w, py * h
                        p2x, p2y = pts_norm[i + 1][0] * w, pts_norm[i + 1][1] * h
                        if matriz_homografia is not None:
                            pts_h = np.array([[[p1x, p1y]], [[p2x, p2y]]], dtype=np.float32)
                            pts_m = cv2.perspectiveTransform(pts_h, matriz_homografia)
                            dm = float(math.hypot(pts_m[1][0][0] - pts_m[0][0][0], pts_m[1][0][1] - pts_m[0][0][1]))
                        else:
                            dm = math.hypot(((p2x - p1x) / w) * 52.0, ((p2y - p1y) / h) * 38.0)
                        mx, my = (p1x + p2x) / 2.0, (p1y + p2y) / 2.0
                        rect_d = QRectF(mx - 28, my - 12, 56, 22)
                        p.setBrush(QBrush(QColor(10, 14, 22, 220)))
                        p.setPen(QPen(QColor(cor_hex), 1.4))
                        p.drawRoundedRect(rect_d, 4, 4)
                        p.setPen(QColor("#FFFFFF"))
                        p.setFont(QFont("SF Mono", 10, QFont.Weight.Bold))
                        p.drawText(rect_d, Qt.AlignmentFlag.AlignCenter, f"{dm:.1f}m")

            elif tp == "seta":
                x1, y1 = ef["nx1"] * w, ef["ny1"] * h
                x2, y2 = ef["nx2"] * w, ef["ny2"] * h
                dist = math.hypot(x2 - x1, y2 - y1)
                elev = min(h * 0.16, dist * 0.28)
                c_ctrl = QPointF((x1 + x2) / 2.0, min(y1, y2) - elev)
                path = QPainterPath(QPointF(x1, y1))
                path.quadTo(c_ctrl, QPointF(x2, y2))
                p.setPen(QPen(QColor(0, 0, 0, 120), 4.0, Qt.PenStyle.DashLine))
                p.drawLine(QPointF(x1, y1), QPointF(x2, y2))
                c_glow = QColor(cor_hex)
                c_glow.setAlpha(130)
                p.setBrush(Qt.BrushStyle.NoBrush)
                p.setPen(QPen(c_glow, 9.0, Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
                p.drawPath(path)
                p.setPen(QPen(QColor(cor_hex), 4.0, Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
                p.drawPath(path)
                ang_f = math.atan2(y2 - c_ctrl.y(), x2 - c_ctrl.x())
                tam_p = 18.0
                ponta = QPolygonF([
                    QPointF(x2, y2),
                    QPointF(x2 - tam_p * math.cos(ang_f - 0.45), y2 - tam_p * math.sin(ang_f - 0.45)),
                    QPointF(x2 - tam_p * math.cos(ang_f + 0.45), y2 - tam_p * math.sin(ang_f + 0.45))
                ])
                p.setBrush(QBrush(QColor("#FFFFFF")))
                p.setPen(QPen(QColor(cor_hex), 2.0))
                p.drawPolygon(ponta)

            elif tp == "livre":
                pts_l = ef.get("pontos", [])
                if len(pts_l) > 1:
                    path_l = QPainterPath(QPointF(pts_l[0][0] * w, pts_l[0][1] * h))
                    for px, py in pts_l[1:]:
                        path_l.lineTo(px * w, py * h)
                    c_glow = QColor(cor_hex)
                    c_glow.setAlpha(140)
                    p.setBrush(Qt.BrushStyle.NoBrush)
                    p.setPen(QPen(c_glow, 7.0, Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
                    p.drawPath(path_l)
                    p.setPen(QPen(QColor("#FFFFFF"), 2.5, Qt.PenStyle.SolidLine, Qt.PenCapStyle.RoundCap))
                    p.drawPath(path_l)

            elif tp == "texto":
                cx, cy = int(ef["nx"] * w), int(ef["ny"] * h)
                larg_box = int((ef.get("largura_pct", 34) / 100.0) * w)
                f_q = QFont(ef.get("fonte", "Helvetica Neue"), int(ef.get("tamanho", 24)))
                f_q.setBold(ef.get("negrito", True))
                f_q.setItalic(ef.get("italico", False))
                p.setFont(f_q)
                align_map = {"Esquerda": Qt.AlignmentFlag.AlignLeft, "Centro": Qt.AlignmentFlag.AlignHCenter, "Direita": Qt.AlignmentFlag.AlignRight}
                flags = align_map.get(ef.get("alinhamento", "Esquerda"), Qt.AlignmentFlag.AlignLeft) | Qt.TextFlag.TextWordWrap
                br = p.boundingRect(QRect(0, 0, max(60, larg_box - 24), h), flags, ef.get("texto", ""))
                alt_min = max(44, br.height() + 24)
                alt_box = max(alt_min, int((ef.get("altura_pct", 0) / 100.0) * h))
                rx = max(10, min(w - larg_box - 10, cx - larg_box // 2))
                ry = max(10, min(h - alt_box - 10, cy - alt_box // 2))
                rect_box = QRectF(rx, ry, larg_box, alt_box)
                ef["_rect_norm"] = (rx / w, ry / h, larg_box / w, alt_box / h)
                opac = int((ef.get("opacidade", 82) / 100.0) * 255)
                if opac > 0 and ef.get("cor_fundo") != "transparent":
                    cor_bg = QColor(ef.get("cor_fundo", "#0B0F19"))
                    cor_bg.setAlpha(opac)
                    p.setBrush(QBrush(cor_bg))
                    p.setPen(QPen(QColor("#38BDF8"), 2.0))
                    p.drawRoundedRect(rect_box, 8.0, 8.0)
                p.setPen(QColor(ef.get("cor_texto", "#FFFFFF")))
                p.drawText(rect_box.adjusted(12, 10, -12, -10), flags, ef.get("texto", ""))

        if pts_calibracao:
            for idx_c, (cx_n, cy_n) in enumerate(pts_calibracao):
                p.setBrush(QBrush(QColor("#22C55E")))
                p.setPen(QPen(QColor("#FFFFFF"), 2.0))
                p.drawEllipse(QPointF(cx_n * w, cy_n * h), 8, 8)
                p.drawText(QPointF(cx_n * w + 10, cy_n * h - 6), f"C{idx_c+1}")

        if legenda_inferior:
            bh = int(h * 0.072)
            by = h - bh - 18
            rect_leg = QRectF(40, by, w - 80, bh)
            p.setBrush(QBrush(QColor(8, 12, 20, 225)))
            p.setPen(Qt.PenStyle.NoPen)
            p.drawRoundedRect(rect_leg, 6, 6)
            p.fillRect(QRectF(40, by, 7, bh), QColor("#00E5FF"))
            p.setPen(QColor("#FFFFFF"))
            p.setFont(QFont("Helvetica Neue", max(14, int(bh * 0.40)), QFont.Weight.Bold))
            p.drawText(rect_leg.adjusted(22, 0, -14, 0), Qt.AlignmentFlag.AlignVCenter | Qt.AlignmentFlag.AlignLeft, legenda_inferior.upper())

        p.end()

        ptr = qimg.constBits()
        ptr.setsize(h * w * 3)
        img = cv2.cvtColor(np.frombuffer(ptr, np.uint8).reshape((h, w, 3)).copy(), cv2.COLOR_RGB2BGR)

        if mostrar_minimapa:
            mw, mh = int(w * 0.18), int(h * 0.16)
            mx1, my1 = w - mw - 20, 20
            rad = np.zeros((mh, mw, 3), dtype=np.uint8)
            rad[:] = (16, 36, 24)
            cv2.rectangle(rad, (2, 2), (mw - 2, mh - 2), (210, 210, 210), 1)
            cv2.line(rad, (mw // 2, 2), (mw // 2, mh - 2), (210, 210, 210), 1)
            for e in efeitos:
                if "nx" in e and "ny" in e:
                    cv2.circle(rad, (int(np.clip(e["nx"] * mw, 5, mw - 5)), int(np.clip(e["ny"] * mh, 5, mh - 5))), 4, (0, 229, 255), -1)
                elif "nx2" in e and "ny2" in e:
                    cv2.circle(rad, (int(np.clip(e["nx2"] * mw, 5, mw - 5)), int(np.clip(e["ny2"] * mh, 5, mh - 5))), 4, (56, 248, 120), -1)
            img[my1:my1+mh, mx1:mx1+mw] = cv2.addWeighted(img[my1:my1+mh, mx1:mx1+mw], 0.18, rad, 0.82, 0)

        return img

    @staticmethod
    def gerar_quadro_tatico_base(w=1280, h=720, titulo="QUADRO TÁTICO", subtitulo=""):
        quadro = np.zeros((h, w, 3), dtype=np.uint8)
        quadro[:] = (14, 34, 22)
        mx, my = 70, 75
        cw, ch = w - 2 * mx, h - 2 * my
        cv2.rectangle(quadro, (mx, my), (mx + cw, my + ch), (220, 220, 220), 2)
        cv2.line(quadro, (w // 2, my), (w // 2, my + ch), (220, 220, 220), 2)
        cv2.circle(quadro, (w // 2, h // 2), int(min(cw, ch) * 0.14), (220, 220, 220), 2)
        cv2.putText(quadro, titulo.upper(), (mx, 45), cv2.FONT_HERSHEY_SIMPLEX, 1.0, (255, 255, 255), 2, cv2.LINE_AA)
        return quadro

    def atualizar_preview_estudio(self, laser_coord=None, efeito_preview=None, idx_selecionado=None):
        kf = self.obter_keyframe_studio_ativo()
        if kf is None or self.frame_studio_original is None:
            return
        lista_fx = list(kf.get("efeitos", []))
        ef_rasc = efeito_preview if efeito_preview is not None else self.canvas_studio.efeito_rascunho
        if ef_rasc is not None:
            lista_fx.append(ef_rasc)

        img_bgr = self.desenhar_efeitos_broadcast(
            self.frame_studio_original, lista_fx,
            self.chk_minimapa.isChecked(),
            self.chk_mask_3d.isChecked(),
            legenda_inferior=kf.get("nota", ""),
            masc_relva_precalc=getattr(self, "masc_relva_cache", None),
            matriz_homografia=getattr(self, "matriz_homografia", None),
            pts_calibracao=getattr(self, "pontos_calibracao", None)
        )

        ef_handles = ef_rasc if ef_rasc is not None else (
            kf["efeitos"][idx_selecionado] if (idx_selecionado is not None and 0 <= idx_selecionado < len(kf.get("efeitos", []))) else None
        )
        if ef_handles is not None:
            h, w = img_bgr.shape[:2]
            if ef_handles.get("tipo") == "texto" and "_rect_norm" in ef_handles:
                rx, ry, rw, rh = ef_handles["_rect_norm"]
                x1, y1, x2, y2 = int(rx * w), int(ry * h), int((rx + rw) * w), int((ry + rh) * h)
                cv2.rectangle(img_bgr, (x1, y1), (x2, y2), (0, 215, 255), 2, cv2.LINE_AA)
                for pt in ((x1, (y1 + y2) // 2), (x2, (y1 + y2) // 2), ((x1 + x2) // 2, y1), ((x1 + x2) // 2, y2)):
                    cv2.circle(img_bgr, pt, 6, (0, 215, 255), -1, cv2.LINE_AA)
            elif "nx" in ef_handles and "ny" in ef_handles:
                cx, cy = int(ef_handles["nx"] * w), int(ef_handles["ny"] * h)
                cv2.circle(img_bgr, (cx, cy), 9, (0, 215, 255), 2, cv2.LINE_AA)
                cv2.circle(img_bgr, (cx, cy), 4, (255, 255, 255), -1, cv2.LINE_AA)
            elif "nx1" in ef_handles and "nx2" in ef_handles:
                p1 = (int(ef_handles["nx1"] * w), int(ef_handles["ny1"] * h))
                p2 = (int(ef_handles["nx2"] * w), int(ef_handles["ny2"] * h))
                pm = ((p1[0] + p2[0]) // 2, (p1[1] + p2[1]) // 2)
                for pt in (p1, p2, pm):
                    cv2.circle(img_bgr, pt, 8, (0, 215, 255), -1, cv2.LINE_AA)
                    cv2.circle(img_bgr, pt, 4, (255, 255, 255), -1, cv2.LINE_AA)

        img_rgb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB)
        h, w, ch = img_rgb.shape
        qimg = QImage(img_rgb.data, w, h, ch * w, QImage.Format.Format_RGB888)
        dpr = self.devicePixelRatioF()
        pix = QPixmap.fromImage(qimg).scaled(
            int(max(400, self.canvas_studio.width()) * dpr),
            int(max(300, self.canvas_studio.height()) * dpr),
            Qt.AspectRatioMode.KeepAspectRatio,
            Qt.TransformationMode.FastTransformation
        )
        pix.setDevicePixelRatio(dpr)
        self.canvas_studio.setPixmap(pix)
        if hasattr(self, "studio_nle_timeline"):
            self.studio_nle_timeline.update()

    def obter_frame_renderizado_clip(self, clip):
        if not os.path.exists(self.caminho_video):
            return None
        kfs = clip.get("keyframes", [])
        t_alvo = kfs[0]["tempo"] if kfs else clip["momento"]
        efeitos_alvo = kfs[0]["efeitos"] if kfs else clip.get("efeitos", [])
        nota_alvo = kfs[0].get("nota", "") if kfs else clip.get("nota", "")

        cap = cv2.VideoCapture(self.caminho_video)
        cap.set(cv2.CAP_PROP_POS_FRAMES, int(t_alvo * self.fps))
        ret, frame = cap.read()
        cap.release()
        if not ret:
            return None
        return self.desenhar_efeitos_broadcast(frame, efeitos_alvo, self.chk_minimapa.isChecked(), self.chk_mask_3d.isChecked(), nota_alvo, matriz_homografia=self.matriz_homografia)

    def exportar_relatorio_pdf_balneario(self):
        if not self.studio_clips:
            QMessageBox.warning(self, "Aviso", "Sincronize primeiro cortes no Vision Studio para gerar o PDF.")
            return

        caminho_pdf, _ = QFileDialog.getSaveFileName(
            self, "Guardar Relatório Tático PDF",
            os.path.expanduser("~/Desktop/KRONOS_Dossier_Tatico.pdf"),
            "Ficheiro PDF (*.pdf)"
        )
        if not caminho_pdf:
            return

        writer = QPdfWriter(caminho_pdf)
        writer.setPageSize(QPageSize(QPageSize.PageSizeId.A4))
        writer.setPageOrientation(QPageLayout.Orientation.Landscape)
        writer.setResolution(300)

        painter = QPainter(writer)
        painter.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        painter.setRenderHint(QPainter.RenderHint.TextAntialiasing, True)

        pw, ph = writer.width(), writer.height()

        for idx, clip in enumerate(self.studio_clips):
            if idx > 0:
                writer.newPage()

            painter.fillRect(0, 0, pw, ph, QColor("#07070C"))
            painter.fillRect(0, 0, pw, int(ph * 0.11), QColor("#0F172A"))

            painter.setPen(QColor("#F59E0B"))
            painter.setFont(QFont("Helvetica Neue", 14, QFont.Weight.Bold))
            painter.drawText(int(pw * 0.04), int(ph * 0.065), f"KRONOS PRO  •  DOSSIER TÁTICO DE JOGO   [{idx+1}/{len(self.studio_clips)}]")

            painter.setPen(QColor("#FFFFFF"))
            painter.setFont(QFont("Helvetica Neue", 16, QFont.Weight.Bold))
            t_info = f"{clip['acao'].upper()}   ({self.formatar_tempo(clip['inicio'], False)} – {self.formatar_tempo(clip['fim'], False)})"
            painter.drawText(int(pw * 0.48), int(ph * 0.065), t_info)

            frame_bgr = self.obter_frame_renderizado_clip(clip)
            if frame_bgr is not None:
                rgb = cv2.cvtColor(frame_bgr, cv2.COLOR_BGR2RGB)
                ih, iw, ch = rgb.shape
                qimg = QImage(rgb.data, iw, ih, ch * iw, QImage.Format.Format_RGB888)
                rect_destino = QRect(int(pw * 0.04), int(ph * 0.14), int(pw * 0.92), int(ph * 0.68))
                painter.drawImage(rect_destino, qimg)

            y_rodape = int(ph * 0.86)
            painter.fillRect(int(pw * 0.04), y_rodape, int(pw * 0.92), int(ph * 0.10), QColor("#11111A"))
            painter.setPen(QColor("#38BDF8"))
            painter.setFont(QFont("Helvetica Neue", 11, QFont.Weight.Bold))
            etiquetas_str = ", ".join(clip.get("labels", [])) or "Sem etiquetas"
            painter.drawText(int(pw * 0.06), y_rodape + int(ph * 0.04), f"Zona: {clip.get('zona', 'N/A')}   |   Etiquetas: {etiquetas_str}")

            painter.setPen(QColor("#F8FAFC"))
            painter.setFont(QFont("Helvetica Neue", 12, QFont.Weight.Normal))
            nota_str = clip.get("nota", "") or "Análise visual gerada no Vision Studio."
            painter.drawText(int(pw * 0.06), y_rodape + int(ph * 0.08), f"Observação Tática: {nota_str}")

        painter.end()
        QMessageBox.information(self, "Relatório PDF Criado", f"Dossier Tático PDF exportado com sucesso:\n{caminho_pdf}")

    def iniciar_exportacao_mp4(self, modo_rapido=False):
        lista_exportar = self.studio_clips if (not modo_rapido and self.studio_clips) else self.eventos_registados
        if not lista_exportar:
            QMessageBox.warning(self, "Aviso", "Sem cortes registados para exportar.")
            return
        if not os.path.exists(self.caminho_video):
            QMessageBox.warning(self, "Aviso", "Carregue primeiro o ficheiro de vídeo original antes de exportar.")
            return

        caminho_saida, _ = QFileDialog.getSaveFileName(
            self,
            "Guardar Vídeo Compilado (.MP4)",
            os.path.expanduser("~/Desktop/KRONOS_Compilacao.mp4"),
            "Vídeo MP4 (*.mp4)"
        )
        if not caminho_saida:
            return
        if not caminho_saida.lower().endswith(".mp4"):
            caminho_saida += ".mp4"

        self.media_player.pause()
        self.btn_play.definir_modo("play")
        if hasattr(self, "btn_play_studio"):
            self.btn_play_studio.definir_modo("play")

        self.dlg_progresso = QProgressDialog("A iniciar motor de compilação...", None, 0, 100, self)
        self.dlg_progresso.setWindowTitle("Exportação de Vídeo KRONOS")
        self.dlg_progresso.setWindowModality(Qt.WindowModality.WindowModal)
        self.dlg_progresso.setMinimumDuration(0)
        self.dlg_progresso.setValue(5)

        self.worker_export = VideoExportWorker(
            self, self.caminho_video, lista_exportar, caminho_saida,
            self.chk_title_card.isChecked(), self.chk_minimapa.isChecked(),
            self.chk_mask_3d.isChecked(), self.fps, modo_rapido_ffmpeg=modo_rapido
        )
        self.worker_export.progresso.connect(self.ao_atualizar_progresso_export)
        self.worker_export.concluido.connect(self.ao_concluir_exportacao)
        self.worker_export.erro.connect(lambda msg: (self.dlg_progresso.close(), QMessageBox.critical(self, "Erro", msg)))
        self.worker_export.start()

    def ao_atualizar_progresso_export(self, val, texto):
        self.dlg_progresso.setLabelText(texto)
        self.dlg_progresso.setValue(val)

    def ao_concluir_exportacao(self, caminho):
        self.dlg_progresso.close()
        self.limpar_autosave()
        QMessageBox.information(self, "Exportação Concluída", f"Ficheiro MP4 guardado na Secretária:\n{caminho}")

    def desenhar_heatmap_espacial(self):
        if not hasattr(self, "lbl_heatmap_canvas"):
            return
        mod = getattr(self, "modalidade_atual", "futebol")
        w, h = 920, 560
        campo_qimg = QImage(w, h, QImage.Format.Format_RGB888)
        cores_envolvente = {
            "voleibol": QColor("#081E46"),
            "basquetebol": QColor("#1E3A8A"),
            "andebol": QColor("#07131C"),
            "futebol": QColor("#050508")
        }
        campo_qimg.fill(cores_envolvente.get(mod, QColor("#050508")))

        marg_x, marg_y = 48, 44
        cw, ch = w - 2 * marg_x, h - 2 * marg_y
        rect_campo = QRectF(marg_x, marg_y, cw, ch)

        filtro_acao = self.combo_heatmap.currentText() if hasattr(self, "combo_heatmap") else "Todas as Ações"
        filtro_lbl = self.combo_etiqueta_spatial.currentText() if hasattr(self, "combo_etiqueta_spatial") else "Todas as Etiquetas"

        evs = [
            e for e in self.eventos_registados
            if (filtro_acao == "Todas as Ações" or e.get("acao") == filtro_acao)
            and (filtro_lbl == "Todas as Etiquetas" or filtro_lbl in e.get("labels", []))
        ]

        usar_calor = self.chk_spatial_heatmap.isChecked() if hasattr(self, "chk_spatial_heatmap") else True
        usar_pontos = self.chk_spatial_pontos.isChecked() if hasattr(self, "chk_spatial_pontos") else True
        usar_zonas = self.chk_spatial_zonas.isChecked() if hasattr(self, "chk_spatial_zonas") else True

        p = QPainter(campo_qimg)
        p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
        p.setRenderHint(QPainter.RenderHint.TextAntialiasing, True)

        self._desenhar_campo_por_modalidade(p, mod, rect_campo, mostrar_zonas=usar_zonas)

        acum = np.zeros((h, w), dtype=np.float32)
        contagem_zonas = {}

        for e in evs:
            cx_n, cy_n = e.get("coord", (0.5, 0.5))
            cx_n = max(0.0, min(1.0, float(cx_n)))
            cy_n = max(0.0, min(1.0, float(cy_n)))
            px = int(round(marg_x + cx_n * cw))
            py = int(round(marg_y + cy_n * ch))
            if usar_calor:
                cv2.circle(acum, (px, py), 62, 1.0, -1)

            zona = e.get("zona")
            if not zona:
                if mod == "voleibol":
                    zona, _ = self._calcular_zona_voleibol(cx_n, cy_n)
                elif mod == "basquetebol":
                    zona, _ = self._calcular_zona_basquetebol(cx_n, cy_n)
                elif mod == "andebol":
                    zona, _ = self._calcular_zona_andebol(cx_n, cy_n)
                else:
                    col = min(5, int(cx_n * 6))
                    row = min(2, int(cy_n * 3))
                    corredores = ["Corredor Esq", "Corredor Central", "Corredor Dir"]
                    zona = f"Zona {col * 3 + row + 1} ({corredores[row]})"
            contagem_zonas[zona] = contagem_zonas.get(zona, 0) + 1

        if usar_calor and np.max(acum) > 0:
            p.end()
            ptr = campo_qimg.constBits()
            ptr.setsize(h * w * 3)
            campo_bgr = cv2.cvtColor(np.frombuffer(ptr, np.uint8).reshape((h, w, 3)).copy(), cv2.COLOR_RGB2BGR)
            acum_blur = cv2.GaussianBlur(acum, (95, 95), 0)
            norm = (acum_blur / np.max(acum_blur) * 255).astype(np.uint8)
            cmap = cv2.applyColorMap(norm, cv2.COLORMAP_JET)
            masc_campo = np.zeros((h, w), dtype=np.uint8)
            cv2.rectangle(masc_campo, (marg_x, marg_y), (marg_x + cw, marg_y + ch), 255, -1)
            ativo_px = ((norm > 12) & (masc_campo > 0))[:, :, np.newaxis]
            fusao = cv2.addWeighted(campo_bgr, 0.42, cmap, 0.58, 0)
            campo_bgr = np.where(ativo_px, fusao, campo_bgr)
            rgb = cv2.cvtColor(campo_bgr, cv2.COLOR_BGR2RGB)
            campo_qimg = QImage(rgb.data, w, h, 3 * w, QImage.Format.Format_RGB888).copy()
            p = QPainter(campo_qimg)
            p.setRenderHint(QPainter.RenderHint.Antialiasing, True)
            self._desenhar_campo_por_modalidade(p, mod, rect_campo, mostrar_zonas=usar_zonas)

        if usar_pontos:
            for e in evs:
                cx_n, cy_n = e.get("coord", (0.5, 0.5))
                px = marg_x + max(0.0, min(1.0, float(cx_n))) * cw
                py = marg_y + max(0.0, min(1.0, float(cy_n))) * ch
                cor_hex = self.todas_pistas.get(e.get("acao", ""), {}).get("cor", "#F59E0B")
                p.setBrush(QBrush(QColor("#000000")))
                p.setPen(Qt.PenStyle.NoPen)
                p.drawEllipse(QPointF(px, py), 7.0, 7.0)
                p.setBrush(QBrush(QColor(cor_hex)))
                p.setPen(QPen(QColor("#FFFFFF"), 1.2))
                p.drawEllipse(QPointF(px, py), 5.0, 5.0)

        p.end()

        dpr = self.devicePixelRatioF()
        pix = QPixmap.fromImage(campo_qimg).scaled(
            int(max(400, self.lbl_heatmap_canvas.width()) * dpr),
            int(max(300, self.lbl_heatmap_canvas.height()) * dpr),
            Qt.AspectRatioMode.KeepAspectRatio,
            Qt.TransformationMode.SmoothTransformation
        )
        pix.setDevicePixelRatio(dpr)
        self.lbl_heatmap_canvas.setPixmap(pix)

        total_evs = len(evs)
        resumo = (
            f"RELATÓRIO ZONAL ({mod.upper()})\n"
            f"Ação: {filtro_acao}\n"
            f"Etiqueta: {filtro_lbl}\n"
            f"Registos: {total_evs}\n"
            + ("-" * 30) + "\n\n"
        )
        if not contagem_zonas:
            resumo += "Sem registos para os filtros selecionados.\n"
        else:
            for z, q in sorted(contagem_zonas.items(), key=lambda x: x[1], reverse=True):
                pct = (q / max(1, total_evs)) * 100.0
                resumo += f"{z}: {q} ({pct:.1f}%)\n"
        if hasattr(self, "txt_resumo_zonas"):
            self.txt_resumo_zonas.setText(resumo)

    def guardar_projeto_json(self):
        caminho, _ = QFileDialog.getSaveFileName(self, "Guardar Sessão Completa", "sessao_kronos.json", "JSON (*.json)")
        if caminho:
            dados = {
                "modalidade_atual": getattr(self, "modalidade_atual", "futebol"),
                "caminho_video": self.caminho_video,
                "layout_items": self.layout_items,
                "eventos_registados": self.eventos_registados,
                "studio_clips": self.studio_clips
            }
            with open(caminho, "w", encoding="utf-8") as f:
                json.dump(dados, f, ensure_ascii=False, indent=2)
            self.limpar_autosave()

    def abrir_projeto_json(self):
        caminho, _ = QFileDialog.getOpenFileName(self, "Importar Sessão", "", "JSON (*.json)")
        if caminho:
            with open(caminho, "r", encoding="utf-8") as f:
                dados = json.load(f)
            mod_s = dados.get("modalidade_atual")
            if mod_s in ("futebol", "voleibol", "basquetebol", "andebol"):
                self.modalidade_atual = mod_s
                if hasattr(self, "combo_modalidade"):
                    idx_m = self.combo_modalidade.findData(mod_s)
                    if idx_m >= 0:
                        self.combo_modalidade.blockSignals(True)
                        self.combo_modalidade.setCurrentIndex(idx_m)
                        self.combo_modalidade.blockSignals(False)
            if "layout_items" in dados:
                self.layout_items = dados["layout_items"]
                self.campo_oculto = not any(it.get("tipo") == "minicampo" for it in self.layout_items)
                self.freeform_canvas.reconstruir_widgets()
                self.sincronizar_estruturas_desde_canvas()
            self.eventos_registados = dados.get("eventos_registados", [])
            self.studio_clips = dados.get("studio_clips", [])
            self.atualizar_dashboard()
            self.atualizar_lista_ui_estudio()
            self.timeline_widget.update()

    def exportar_excel(self):
        if not self.eventos_registados:
            return
        caminho, _ = QFileDialog.getSaveFileName(
            self, "Exportar Dados (CSV)",
            os.path.expanduser("~/Desktop/kronos_eventos.csv"),
            "CSV (*.csv)"
        )
        if not caminho:
            return
        with open(caminho, mode="w", newline="", encoding="utf-8") as f:
            w = csv.writer(f)
            w.writerow(["Categoria", "Início", "Fim", "Duração", "Zona", "Etiquetas", "Nota"])
            for ev in self.eventos_registados:
                w.writerow([ev["acao"], ev["inicio"], ev["fim"], round(ev["fim"] - ev["inicio"], 2), ev.get("zona", ""), ", ".join(ev["labels"]), ev["nota"]])
        self.limpar_autosave()
        QMessageBox.information(self, "Exportado", f"Ficheiro CSV guardado na Secretária:\n{caminho}")

    def _garantir_pastas_kronos(self):
        for _p in (PASTA_PROJETOS, PASTA_EXPORTACOES, PASTA_AUTOSAVE):
            _p.mkdir(parents=True, exist_ok=True)

    def _nome_sugerido_projeto(self):
        base_video = Path(self.caminho_video).stem if self.caminho_video else "jogo_analise"
        data_str = time.strftime("%Y-%m-%d")
        limpo = "".join(c if c.isalnum() or c in ("-", "_") else "_" for c in base_video).strip("_")
        return f"{data_str}_{limpo or 'projeto'}.kronos.json"

    def registar_projeto_recente(self, caminho_projeto):
        if not caminho_projeto:
            return
        self._garantir_pastas_kronos()
        recentes = []
        if RECENTES_PATH.exists():
            try:
                with open(RECENTES_PATH, "r", encoding="utf-8") as f:
                    recentes = json.load(f)
            except Exception:
                recentes = []

        entrada = {
            "projeto": str(caminho_projeto),
            "nome": Path(caminho_projeto).name,
            "video": self.caminho_video,
            "modalidade": getattr(self, "modalidade_atual", "futebol"),
            "atualizado_em": time.strftime("%Y-%m-%d %H:%M"),
        }
        recentes = [r for r in recentes if r.get("projeto") != str(caminho_projeto)]
        recentes.insert(0, entrada)
        recentes = recentes[:12]

        try:
            with open(RECENTES_PATH, "w", encoding="utf-8") as f:
                json.dump(recentes, f, ensure_ascii=False, indent=2)
        except Exception as e:
            print(f"Erro ao atualizar recentes: {e}")

    def abrir_pasta_kronos(self):
        self._garantir_pastas_kronos()
        destino = str(PASTA_KRONOS_BASE)
        try:
            if platform.system() == "Darwin":
                subprocess.run(["open", destino], check=False)
            elif platform.system() == "Windows":
                os.startfile(destino)  # type: ignore[attr-defined]
            else:
                subprocess.run(["xdg-open", destino], check=False)
        except Exception as e:
            QMessageBox.information(self, "Pasta Kronos Pro", f"Os seus projetos e exportações estão em:\n{destino}\n\n({e})")

    def abrir_projetos_recentes_dialog(self):
        recentes = []
        if RECENTES_PATH.exists():
            try:
                with open(RECENTES_PATH, "r", encoding="utf-8") as f:
                    recentes = json.load(f)
            except Exception:
                recentes = []

        validos = [r for r in recentes if r.get("projeto") and os.path.exists(r["projeto"])]
        if not validos:
            QMessageBox.information(self, "Projetos Recentes", "Ainda não existem projetos guardados recentemente.")
            return

        opcoes = [
            f"{r.get('nome')}  |  {r.get('modalidade', '').upper()}  |  {r.get('atualizado_em', '')}"
            for r in validos
        ]
        escolha, ok = QInputDialog.getItem(self, "Projetos Recentes — KRONOS PRO", "Escolha um projeto para abrir:", opcoes, 0, False)
        if ok and escolha:
            idx = opcoes.index(escolha)
            self._carregar_dados_projeto(validos[idx]["projeto"])

    def _carregar_dados_projeto(self, caminho):
        with open(caminho, "r", encoding="utf-8") as f:
            dados = json.load(f)

        mod_s = dados.get("modalidade_atual")
        if mod_s in ("futebol", "voleibol", "basquetebol", "andebol"):
            self.modalidade_atual = mod_s
            if hasattr(self, "combo_modalidade"):
                idx_m = self.combo_modalidade.findData(mod_s)
                if idx_m >= 0:
                    self.combo_modalidade.blockSignals(True)
                    self.combo_modalidade.setCurrentIndex(idx_m)
                    self.combo_modalidade.blockSignals(False)

        if "layout_items" in dados:
            self.layout_items = dados["layout_items"]
            self.campo_oculto = not any(it.get("tipo") == "minicampo" for it in self.layout_items)
            if hasattr(self, "freeform_canvas"):
                self.freeform_canvas.reconstruir_widgets()
            self.sincronizar_estruturas_desde_canvas()

        self.eventos_registados = dados.get("eventos_registados", [])
        self.studio_clips = dados.get("studio_clips", [])
        self.projeto_atual_path = str(caminho)
        self.registar_projeto_recente(self.projeto_atual_path)

        vid = dados.get("caminho_video", "")
        if vid and os.path.exists(vid):
            self.caminho_video = vid
            self.iniciar_video()
        elif vid:
            self.caminho_video = vid
            resp = QMessageBox.question(
                self,
                "Vídeo Não Encontrado",
                f"O vídeo original não foi encontrado no caminho guardado:\n{vid}\n\nDeseja localizar o ficheiro de vídeo agora?",
                QMessageBox.StandardButton.Yes | QMessageBox.StandardButton.No,
                QMessageBox.StandardButton.Yes,
            )
            if resp == QMessageBox.StandardButton.Yes:
                self.escolher_ficheiro_video()

        self.atualizar_dashboard()
        self.atualizar_lista_ui_estudio()
        self.timeline_widget.update()

    def guardar_autosave(self):
        try:
            dados = {
                "modalidade_atual": getattr(self, "modalidade_atual", "futebol"),
                "caminho_video": self.caminho_video,
                "eventos_registados": self.eventos_registados,
                "studio_clips": self.studio_clips,
            }
            with open(AUTOSAVE_PATH, "w", encoding="utf-8") as f:
                json.dump(dados, f, ensure_ascii=False, indent=2)
            self.alteracoes_por_guardar = True
        except Exception as e:
            print(f"Erro no auto-save: {e}")

    def verificar_recuperacao(self):
        if not AUTOSAVE_PATH.exists():
            return
        try:
            with open(AUTOSAVE_PATH, "r", encoding="utf-8") as f:
                dados = json.load(f)
        except Exception:
            return

        evs = dados.get("eventos_registados", [])
        if not evs and not dados.get("studio_clips"):
            return

        resposta = QMessageBox.question(
            self,
            "Recuperar Sessão Anterior",
            f"Foi encontrada uma sessão não guardada com {len(evs)} corte(s).\n\nDeseja recuperar estes dados agora?",
            QMessageBox.StandardButton.Yes | QMessageBox.StandardButton.No,
            QMessageBox.StandardButton.Yes,
        )
        if resposta == QMessageBox.StandardButton.Yes:
            mod_s = dados.get("modalidade_atual")
            if mod_s in ("futebol", "voleibol", "basquetebol", "andebol") and mod_s != self.modalidade_atual:
                idx_m = self.combo_modalidade.findData(mod_s)
                if idx_m >= 0:
                    self.combo_modalidade.setCurrentIndex(idx_m)
            self.eventos_registados = evs
            self.studio_clips = dados.get("studio_clips", [])
            if dados.get("caminho_video") and not self.caminho_video:
                self.caminho_video = dados["caminho_video"]
            self.alteracoes_por_guardar = True
            self.atualizar_dashboard()
            self.atualizar_lista_ui_estudio()
            self.timeline_widget.update()
        else:
            self.limpar_autosave()

    def limpar_autosave(self):
        self.alteracoes_por_guardar = False
        if AUTOSAVE_PATH.exists():
            try:
                AUTOSAVE_PATH.unlink()
            except Exception:
                pass

    def closeEvent(self, event):
        if getattr(self, "alteracoes_por_guardar", False) and (
            len(self.eventos_registados) > 0 or len(self.studio_clips) > 0
        ):
            resposta = QMessageBox.question(
                self,
                "Sair do KRONOS PRO",
                "Existem cortes ou alterações por guardar/exportar.\n\nTem a certeza de que pretende fechar a aplicação?",
                QMessageBox.StandardButton.Yes | QMessageBox.StandardButton.No,
                QMessageBox.StandardButton.No,
            )
            if resposta == QMessageBox.StandardButton.No:
                event.ignore()
                return
        event.accept()


if __name__ == "__main__":
    app = QApplication(sys.argv)
    app.setWindowIcon(gerar_icone_app_k(256))
    lic_mgr = LicenseManager()
    estado_lic, dias_rest, _ = lic_mgr.verificar_validade()

    if estado_lic in ("nao_ativado", "expirado_offline"):
        dlg_lic = AtivacaoLicencaDialog(lic_mgr, aviso_expirado=(estado_lic == "expirado_offline"))
        if not dlg_lic.exec():
            sys.exit(0)
    elif estado_lic == "aviso_offline":
        QMessageBox.information(
            None, f"{APP_NAME} — Validação Offline",
            f"Restam {dias_rest} dia(s) de utilização offline. Ligue-se à internet em breve para renovar a sua licença."
        )

    window = KronosProQtApp(lic_mgr)
    window.show()
    sys.exit(app.exec())

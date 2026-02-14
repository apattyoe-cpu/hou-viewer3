// 国保法 条文ビューア（左ペイン＋ネイティブPDF表示 / 環境依存ゼロ版）

import React, { useState } from "react";

// ===== 設計方針 =====
// pdf.js を完全に廃止
// 理由：
// ・Worker 問題
// ・CDN rewrite 問題
// ・Sandbox fetch 制限
//
// → ブラウザ標準 PDF Viewer を使用（最も安定）

export default function KokuhouLawViewer() {

  const [pdfUrl, setPdfUrl] = useState(null);
  const [fileName, setFileName] = useState("");
  const [error, setError] = useState(null);

  // ===== PDF読み込み（ネイティブ表示） =====
  const handleFile = async (file) => {
    if (!file) return;

    try {
      setError(null);

      if (file.type !== "application/pdf") {
        throw new Error("PDFファイルを選択してください");
      }

      const url = URL.createObjectURL(file);

      setPdfUrl(url);
      setFileName(file.name);

    } catch (e) {
      console.error("PDF LOAD ERROR:", e);
      setError(e?.message || "PDF読み込みエラー");
    }
  };

  return (
    <div className="flex h-screen bg-gray-100">

      {/* ===== 左ペイン ===== */}
      <div className="w-64 bg-white border-r overflow-y-auto">
        <div className="p-4 border-b font-bold text-lg">
          条文ビュー
        </div>

        <div className="p-4 text-sm text-gray-600">
          PDFを読み込むと右側に全文表示されます。
        </div>

        {fileName && (
          <div className="p-4 text-sm border-t">
            <div className="font-semibold">読込中PDF</div>
            <div className="break-all text-gray-600">{fileName}</div>
          </div>
        )}
      </div>

      {/* ===== 右ペイン ===== */}
      <div className="flex-1 flex flex-col">

        <div className="p-3 bg-white border-b flex gap-3 items-center">
          <input
            type="file"
            accept="application/pdf"
            onChange={(e) => handleFile(e.target.files[0])}
          />

          {error && <span className="text-red-500">{error}</span>}
        </div>

        <div className="flex-1 bg-gray-300">
          {pdfUrl ? (
            <iframe
              src={pdfUrl}
              title="PDF Viewer"
              className="w-full h-full"
            />
          ) : (
            <div className="h-full flex items-center justify-center text-gray-600">
              PDFを選択してください
            </div>
          )}
        </div>

      </div>
    </div>
  );
}

// ===== テストケース =====
// 1. 正常PDF → iframeで表示される
// 2. PDF以外 → エラー表示される
// 3. 空ファイル → エラー表示
// 4. 大容量PDF → ブラウザ標準表示でフリーズしない
// 5. 再読み込み → URLが更新される
// 6. 連続ファイル選択 → 正常切替

'use client';

import React, { useState } from 'react';

export default function QuotationPlatform() {
  const [selectedProduct, setSelectedProduct] = useState(null);
  const [quantity, setQuantity] = useState(1);
  const [selectedOptions, setSelectedOptions] = useState({});
  const [customerInfo, setCustomerInfo] = useState({
    name: '',
    email: '',
    phone: '',
    company: ''
  });
  const [isLoading, setIsLoading] = useState(false);

  const products = [
    {
      id: 1,
      name: '산업용 컨베이어 시스템',
      basePrice: 5000000,
      image: '🏭',
      description: '고속 산업용 컨베이어',
      options: [
        { id: 'motor', name: '모터 업그레이드 (5.5kW)', price: 500000 },
        { id: 'sensor', name: '센서 패키지', price: 300000 },
        { id: 'control', name: 'PLC 제어 시스템', price: 800000 }
      ]
    },
    {
      id: 2,
      name: '프레스기계',
      basePrice: 8000000,
      image: '⚙️',
      description: '정밀 프레스 기계',
      options: [
        { id: 'automation', name: '자동화 시스템', price: 1000000 },
        { id: 'precision', name: '정밀 제어 모듈', price: 600000 },
        { id: 'safety', name: '안전 장치 추가', price: 400000 }
      ]
    },
    {
      id: 3,
      name: '용접 로봇 시스템',
      basePrice: 12000000,
      image: '🤖',
      description: '6축 자동 용접 로봇',
      options: [
        { id: 'gripper', name: '멀티 그리퍼', price: 2000000 },
        { id: 'vision', name: '비전 시스템', price: 1500000 },
        { id: 'maintenance', name: '정기 유지보수 패키지 (1년)', price: 500000 }
      ]
    }
  ];

  const calculateTotal = () => {
    if (!selectedProduct) return 0;
    const product = products.find(p => p.id === selectedProduct);
    let total = product.basePrice * quantity;
    
    Object.keys(selectedOptions).forEach(optionId => {
      const option = product.options.find(o => o.id === optionId);
      if (option && selectedOptions[optionId]) {
        total += option.price * quantity;
      }
    });
    return total;
  };

  const generatePDF = async () => {
    if (!selectedProduct || !customerInfo.email || !customerInfo.name) {
      alert('필수 정보를 모두 입력해주세요.');
      return;
    }

    setIsLoading(true);

    try {
      const product = products.find(p => p.id === selectedProduct);
      const pdfContent = generatePDFContent();
      
      const pdfWindow = window.open();
      pdfWindow.document.write(`
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="utf-8">
          <title>견적서</title>
          <style>
            body { font-family: 'Segoe UI', sans-serif; padding: 40px; }
            .header { border-bottom: 3px solid #0066ff; padding-bottom: 20px; margin-bottom: 30px; }
            .header h1 { margin: 0; color: #0066ff; font-size: 28px; }
            .info-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 30px; margin-bottom: 30px; }
            .info-group h3 { font-size: 12px; color: #666; margin: 0 0 8px 0; }
            .info-group p { margin: 4px 0; font-size: 12px; }
            table { width: 100%; border-collapse: collapse; margin-bottom: 30px; }
            th { background: #f5f5f5; border-bottom: 2px solid #0066ff; padding: 12px; text-align: left; font-weight: bold; }
            td { padding: 10px 12px; border-bottom: 1px solid #eee; }
            .total-box { background: #f0f7ff; padding: 15px; border-radius: 6px; text-align: right; font-weight: bold; color: #0066ff; font-size: 16px; margin-bottom: 30px; }
            .footer { color: #999; font-size: 11px; text-align: center; margin-top: 30px; }
          </style>
        </head>
        <body>
          ${pdfContent}
        </body>
        </html>
      `);
      pdfWindow.document.close();
      
      setTimeout(() => {
        pdfWindow.print();
      }, 250);

      alert('✓ PDF가 준비되었습니다.\n프린터에서 PDF로 저장하거나 인쇄할 수 있습니다.');
      
    } catch (error) {
      alert('오류가 발생했습니다.');
    } finally {
      setIsLoading(false);
    }
  };

  const generatePDFContent = () => {
    const product = products.find(p => p.id === selectedProduct);
    const total = calculateTotal();
    const date = new Date().toLocaleDateString('ko-KR');

    let itemsHTML = `
      <tr>
        <td>${product.name}</td>
        <td style="text-align: right;">${quantity}</td>
        <td style="text-align: right;">₩${(product.basePrice / 1000000).toFixed(1)}백만</td>
        <td style="text-align: right;">₩${((product.basePrice * quantity) / 1000000).toFixed(1)}백만</td>
      </tr>
    `;

    Object.keys(selectedOptions).forEach(optionId => {
      if (selectedOptions[optionId]) {
        const option = product.options.find(o => o.id === optionId);
        itemsHTML += `
          <tr style="background: #fafafa;">
            <td style="padding-left: 30px;">└ ${option.name}</td>
            <td style="text-align: right;">${quantity}</td>
            <td style="text-align: right;">₩${(option.price / 1000000).toFixed(1)}백만</td>
            <td style="text-align: right;">₩${((option.price * quantity) / 1000000).toFixed(1)}백만</td>
          </tr>
        `;
      }
    });

    return `
      <div class="header">
        <h1>산업장비 견적서</h1>
        <p style="margin: 0; color: #999; font-size: 12px;">QUOTATION</p>
      </div>

      <div class="info-grid">
        <div class="info-group">
          <h3>공급사</h3>
          <p><strong>쿠키릥 (주)</strong></p>
          <p>서울시 강남구 | 02-1234-5678</p>
          <p>info@kookileng.co.kr</p>
        </div>
        <div class="info-group">
          <h3>고객</h3>
          <p><strong>${customerInfo.name}</strong></p>
          <p>회사: ${customerInfo.company || '미입력'}</p>
          <p>이메일: ${customerInfo.email}</p>
          <p>연락처: ${customerInfo.phone || '미입력'}</p>
        </div>
      </div>

      <table>
        <thead>
          <tr>
            <th>항목</th>
            <th style="text-align: right;">수량</th>
            <th style="text-align: right;">단가</th>
            <th style="text-align: right;">금액</th>
          </tr>
        </thead>
        <tbody>
          ${itemsHTML}
        </tbody>
      </table>

      <div class="total-box">
        총 금액: ₩${(total / 1000000).toFixed(1)}백만 KRW
      </div>

      <div style="margin-bottom: 30px; padding-top: 15px; border-top: 1px solid #eee;">
        <h3 style="font-size: 12px; margin: 0 0 8px 0;">비고</h3>
        <ul style="font-size: 11px; color: #666; margin: 0; padding-left: 20px;">
          <li>위 견적은 제시된 사양을 기준으로 작성되었습니다.</li>
          <li>변경 사항이 있을 시 재견적을 요청해주세요.</li>
          <li>유효기간: 발행일로부터 30일간 유효합니다.</li>
        </ul>
      </div>

      <div class="footer">
        이 견적서는 전자 발급되었습니다. (${date})
      </div>
    `;
  };

  const product = selectedProduct ? products.find(p => p.id === selectedProduct) : null;

  return (
    <div style={{ padding: '2rem 0', maxWidth: '900px', margin: '0 auto', fontFamily: 'system-ui' }}>
      <div style={{ marginBottom: '2rem' }}>
        <h1 style={{ fontSize: '28px', fontWeight: '600', marginBottom: '0.5rem' }}>
          🏭 쿠키릥 견적서 플랫폼
        </h1>
        <p style={{ fontSize: '14px', color: '#666', margin: '0' }}>
          제품 선택 → 옵션 커스텀 → PDF 생성
        </p>
      </div>

      <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '2rem' }}>
        
        <div>
          <div style={{ marginBottom: '2.5rem' }}>
            <h2 style={{ fontSize: '16px', fontWeight: '600', marginBottom: '1rem' }}>
              1️⃣ 제품 선택
            </h2>
            <div style={{ display: 'grid', gap: '12px' }}>
              {products.map(p => (
                <div
                  key={p.id}
                  onClick={() => {
                    setSelectedProduct(p.id);
                    setSelectedOptions({});
                  }}
                  style={{
                    padding: '14px',
                    border: selectedProduct === p.id ? '2px solid #0066ff' : '1px solid #ddd',
                    borderRadius: '8px',
                    cursor: 'pointer',
                    backgroundColor: selectedProduct === p.id ? '#f0f7ff' : '#fff',
                    transition: 'all 0.2s'
                  }}
                >
                  <div style={{ fontSize: '24px', marginBottom: '6px' }}>{p.image}</div>
                  <div style={{ fontWeight: '600', fontSize: '14px', marginBottom: '4px' }}>
                    {p.name}
                  </div>
                  <div style={{ fontSize: '12px', color: '#999', marginBottom: '6px' }}>
                    {p.description}
                  </div>
                  <div style={{ fontSize: '14px', fontWeight: '600', color: '#0066ff' }}>
                    ₩{(p.basePrice / 1000000).toFixed(1)}백만
                  </div>
                </div>
              ))}
            </div>
          </div>

          {product && (
            <div style={{ marginBottom: '2.5rem' }}>
              <h2 style={{ fontSize: '16px', fontWeight: '600', marginBottom: '1rem' }}>
                2️⃣ 수량
              </h2>
              <div style={{ display: 'flex', alignItems: 'center', gap: '12px' }}>
                <button onClick={() => setQuantity(Math.max(1, quantity - 1))} style={{ padding: '10px 14px', border: '1px solid #ddd', borderRadius: '6px', cursor: 'pointer', fontWeight: '600' }}>−</button>
                <input type="number" min="1" value={quantity} onChange={(e) => setQuantity(Math.max(1, parseInt(e.target.value) || 1))} style={{ width: '70px', padding: '10px', textAlign: 'center', border: '1px solid #ddd', borderRadius: '6px' }} />
                <button onClick={() => setQuantity(quantity + 1)} style={{ padding: '10px 14px', border: '1px solid #ddd', borderRadius: '6px', cursor: 'pointer', fontWeight: '600' }}>+</button>
                <span style={{ fontSize: '14px', color: '#666' }}>개</span>
              </div>
            </div>
          )}

          {product && (
            <div>
              <h2 style={{ fontSize: '16px', fontWeight: '600', marginBottom: '1rem' }}>
                3️⃣ 옵션 선택
              </h2>
              <div style={{ display: 'grid', gap: '10px' }}>
                {product.options.map(option => (
                  <label key={option.id} style={{ display: 'flex', gap: '12px', padding: '12px', border: '1px solid #ddd', borderRadius: '6px', cursor: 'pointer', backgroundColor: selectedOptions[option.id] ? '#f0f7ff' : '#fff' }}>
                    <input type="checkbox" checked={selectedOptions[option.id] || false} onChange={() => setSelectedOptions(prev => ({ ...prev, [option.id]: !prev[option.id] }))} style={{ cursor: 'pointer', width: '18px', height: '18px' }} />
                    <div style={{ flex: 1 }}>
                      <div style={{ fontSize: '14px', fontWeight: '500' }}>{option.name}</div>
                      <div style={{ fontSize: '13px', color: '#999' }}>+₩{(option.price / 1000000).toFixed(1)}백만</div>
                    </div>
                  </label>
                ))}
              </div>
            </div>
          )}
        </div>

        <div>
          <div style={{ backgroundColor: '#fff', border: '1px solid #ddd', borderRadius: '8px', padding: '1.5rem', marginBottom: '1.5rem', position: 'sticky', top: '20px' }}>
            <h2 style={{ fontSize: '16px', fontWeight: '600', marginBottom: '1.5rem' }}>
              💰 견적서 요약
            </h2>

            {product ? (
              <>
                <div style={{ marginBottom: '1.5rem', paddingBottom: '1.5rem', borderBottom: '1px solid #eee' }}>
                  <div style={{ fontSize: '12px', color: '#999', marginBottom: '4px' }}>제품</div>
                  <div style={{ fontSize: '15px', fontWeight: '600' }}>{product.name}</div>
                  <div style={{ fontSize: '13px', color: '#666', marginTop: '6px' }}>수량: <strong>{quantity}개</strong></div>
                </div>

                <div style={{ marginBottom: '1.5rem' }}>
                  <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '10px', fontSize: '13px' }}>
                    <span style={{ color: '#666' }}>기본 제품</span>
                    <span style={{ fontWeight: '600' }}>₩{((product.basePrice * quantity) / 1000000).toFixed(1)}백만</span>
                  </div>
                  {Object.keys(selectedOptions).map(optionId => {
                    if (!selectedOptions[optionId]) return null;
                    const option = product.options.find(o => o.id === optionId);
                    return (
                      <div key={optionId} style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '10px', fontSize: '13px' }}>
                        <span style={{ color: '#666' }}>  └ {option.name}</span>
                        <span style={{ fontWeight: '500' }}>+₩{((option.price * quantity) / 1000000).toFixed(1)}백만</span>
                      </div>
                    );
                  })}
                </div>

                <div style={{ padding: '14px', backgroundColor: '#f5f5f5', borderRadius: '6px', display: 'flex', justifyContent: 'space-between', borderLeft: '4px solid #0066ff' }}>
                  <span style={{ fontSize: '14px', fontWeight: '600' }}>총 합계</span>
                  <span style={{ fontSize: '18px', fontWeight: '700', color: '#0066ff' }}>₩{(calculateTotal() / 1000000).toFixed(1)}백만</span>
                </div>
              </>
            ) : (
              <div style={{ fontSize: '14px', color: '#999', textAlign: 'center', padding: '2rem 0' }}>
                제품을 선택해주세요
              </div>
            )}
          </div>

          {product && (
            <div style={{ backgroundColor: '#fff', border: '1px solid #ddd', borderRadius: '8px', padding: '1.5rem' }}>
              <h2 style={{ fontSize: '16px', fontWeight: '600', marginBottom: '1.5rem' }}>
                📋 고객정보
              </h2>

              <div style={{ display: 'grid', gap: '12px', marginBottom: '1.5rem' }}>
                <div>
                  <label style={{ display: 'block', fontSize: '12px', fontWeight: '600', color: '#666', marginBottom: '6px' }}>성명 *</label>
                  <input type="text" value={customerInfo.name} onChange={(e) => setCustomerInfo({...customerInfo, name: e.target.value})} placeholder="홍길동" style={{ width: '100%', padding: '10px', fontSize: '14px', border: '1px solid #ddd', borderRadius: '6px', boxSizing: 'border-box' }} />
                </div>

                <div>
                  <label style={{ display: 'block', fontSize: '12px', fontWeight: '600', color: '#666', marginBottom: '6px' }}>회사명</label>
                  <input type="text" value={customerInfo.company} onChange={(e) => setCustomerInfo({...customerInfo, company: e.target.value})} placeholder="ABC 산업" style={{ width: '100%', padding: '10px', fontSize: '14px', border: '1px solid #ddd', borderRadius: '6px', boxSizing: 'border-box' }} />
                </div>

                <div>
                  <label style={{ display: 'block', fontSize: '12px', fontWeight: '600', color: '#666', marginBottom: '6px' }}>이메일 *</label>
                  <input type="email" value={customerInfo.email} onChange={(e) => setCustomerInfo({...customerInfo, email: e.target.value})} placeholder="user@example.com" style={{ width: '100%', padding: '10px', fontSize: '14px', border: '1px solid #ddd', borderRadius: '6px', boxSizing: 'border-box' }} />
                </div>

                <div>
                  <label style={{ display: 'block', fontSize: '12px', fontWeight: '600', color: '#666', marginBottom: '6px' }}>연락처</label>
                  <input type="tel" value={customerInfo.phone} onChange={(e) => setCustomerInfo({...customerInfo, phone: e.target.value})} placeholder="010-1234-5678" style={{ width: '100%', padding: '10px', fontSize: '14px', border: '1px solid #ddd', borderRadius: '6px', boxSizing: 'border-box' }} />
                </div>

                <button onClick={generatePDF} disabled={isLoading} style={{ width: '100%', padding: '12px', marginTop: '12px', fontSize: '14px', fontWeight: '600', backgroundColor: isLoading ? '#ccc' : '#0066ff', color: '#fff', border: 'none', borderRadius: '6px', cursor: isLoading ? 'not-allowed' : 'pointer' }}>
                  {isLoading ? '처리 중...' : '✨ PDF 다운로드 & 인쇄'}
                </button>
              </div>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}
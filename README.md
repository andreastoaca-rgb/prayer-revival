import React, { useState, useEffect, useMemo } from 'react';
import {
  Flame,
  Clock,
  Heart,
  Users,
  Sparkles,
  CheckCircle2,
  Plus,
  User,
  LogOut,
  LogIn,
  Calendar,
  MapPin,
  Shield,
  HandHeart,
  ChevronRight,
  ChevronLeft,
  Info,
  X,
  Play,
  Pause,
  RotateCcw,
  BookOpen,
  CreditCard,
  Building2,
  Wallet,
  Trash2,
  Award,
  Sun,
  Moon,
  ArrowRight,
  Search,
  Filter
} from 'lucide-react';

// --- HILFSFUNKTIONEN FÜR DATUM & WOCHE ---
const getStartOfCurrentWeek = () => {
  const now = new Date();
  const day = now.getDay(); // 0 = So, 1 = Mo, ...
  const diff = now.getDate() - day + (day === 0 ? -6 : 1); // Montag als Wochenstart
  const monday = new Date(now.setDate(diff));
  monday.setHours(0, 0, 0, 0);
  return monday;
};

const formatDateKey = (date) => {
  const y = date.getFullYear();
  const m = String(date.getMonth() + 1).padStart(2, '0');
  const d = String(date.getDate()).padStart(2, '0');
  return `${y}-${m}-${d}`;
};

const getWeekDays = (startDate) => {
  const days = [];
  const dayNames = ['So', 'Mo', 'Di', 'Mi', 'Do', 'Fr', 'Sa'];
  const fullDayNames = ['Sonntag', 'Montag', 'Dienstag', 'Mittwoch', 'Donnerstag', 'Freitag', 'Samstag'];
  const monthNames = ['Jan.', 'Feb.', 'Mär.', 'Apr.', 'Mai', 'Jun.', 'Jul.', 'Aug.', 'Sep.', 'Okt.', 'Nov.', 'Dez.'];

  for (let i = 0; i < 7; i++) {
    const d = new Date(startDate);
    d.setDate(startDate.getDate() + i);
    days.push({
      dateObj: d,
      dateKey: formatDateKey(d),
      shortDay: dayNames[d.getDay()],
      fullDay: fullDayNames[d.getDay()],
      formattedDate: `${d.getDate()}. ${monthNames[d.getMonth()]}`,
      isToday: formatDateKey(d) === formatDateKey(new Date())
    });
  }
  return days;
};

// --- INITIALE BEISPIELDATEN ---
const generateInitialSlots = (weekDays) => {
  const slots = {};
  const samplePrayers = [
    { name: 'Lukas', city: 'Wien', userId: 'demo-1' },
    { name: 'Sarah', city: 'Graz', userId: 'demo-2' },
    { name: 'Markus', city: 'Linz', userId: 'demo-3' },
    { name: 'Hannah', city: 'Salzburg', userId: 'demo-4' },
    { name: 'Tobias', city: 'Innsbruck', userId: 'demo-5' },
    { name: 'Miriam', city: 'St. Pölten', userId: 'demo-6' },
    { name: 'David', city: 'Klagenfurt', userId: 'demo-7' },
    { name: 'Elena', city: 'Bregenz', userId: 'demo-8' },
    { name: 'Jonas', city: 'Eisenstadt', userId: 'demo-9' },
    { name: 'Debora', city: 'Wels', userId: 'demo-10' }
  ];

  // Vorbefüllung für lebendige Atmosphäre (~62% Abdeckung)
  weekDays.forEach((day, dIdx) => {
    for (let h = 0; h < 24; h++) {
      const key = `${day.dateKey}_${h}`;
      const seed = (dIdx * 24 + h * 7) % 13;
      if (seed < 8) {
        const p1 = samplePrayers[(dIdx + h) % samplePrayers.length];
        slots[key] = [{ id: `init-${key}-1`, name: p1.name, city: p1.city, userId: p1.userId }];
        if (seed === 2 || seed === 5) {
          const p2 = samplePrayers[(dIdx + h + 3) % samplePrayers.length];
          slots[key].push({ id: `init-${key}-2`, name: p2.name, city: p2.city, userId: p2.userId });
        }
      } else {
        slots[key] = [];
      }
    }
  });

  // Aktuelle Stunde sicherheitshalber besetzen, damit Live-Anzeige direkt jemanden zeigt
  const now = new Date();
  const currentKey = `${formatDateKey(now)}_${now.getHours()}`;
  if (slots[currentKey] && slots[currentKey].length === 0) {
    slots[currentKey].push({ id: 'live-now', name: 'Sarah', city: 'Graz', userId: 'demo-2' });
  }

  return slots;
};

const INITIAL_REQUESTS = [
  {
    id: 'req-1',
    title: 'Geistlicher Aufbruch an Österreichs Universitäten',
    category: 'Familie & Jugend',
    author: 'Lukas aus Wien',
    date: 'Vor 2 Stunden',
    description: 'Lasst uns für Studierende in Wien, Graz, Linz, Salzburg und Innsbruck beten – dass junge Menschen Jesus persönlich begegnen, Wahrheit finden und brennende Gebetsgruppen auf dem Campus entstehen.',
    prayerCount: 42,
    supporters: ['demo-1', 'demo-2']
  },
  {
    id: 'req-2',
    title: 'Einheit & Feuer unter den Gemeinden in Österreich',
    category: 'Gemeinde',
    author: 'Hannah aus Salzburg',
    date: 'Gestern',
    description: 'Gebet für Pastoren, Leiter und alle christlichen Gemeinden im ganzen Land. Dass konfessionelle Mauern fallen, neue Leidenschaft für das Evangelium ausbricht und Zeichen und Wunder geschehen.',
    prayerCount: 68,
    supporters: ['demo-4']
  },
  {
    id: 'req-3',
    title: 'Weisheit für Regierung und Entscheidungsträger',
    category: 'Erweckung & Land',
    author: 'Markus aus Linz',
    date: 'Vor 2 Tagen',
    description: 'Nach 1. Timotheus 2,1-2 beten wir für alle in Verantwortung in Österreich: für Frieden, Gerechtigkeit, Schutz des Lebens und offene Türen für die Verkündigung der Guten Nachricht.',
    prayerCount: 54,
    supporters: []
  },
  {
    id: 'req-4',
    title: 'Heilung und neue Hoffnung für kranke und einsame Menschen',
    category: 'Heilung',
    author: 'Anonym',
    date: 'Vor 3 Tagen',
    description: 'Viele Menschen in unserem Land leiden im Stillen unter Depressionen, Einsamkeit und schweren Krankheiten. Wir beten für göttlichen Trost, übernatürliche Heilung und Menschen, die ihnen die Liebe Christi bringen.',
    prayerCount: 89,
    supporters: []
  },
  {
    id: 'req-5',
    title: 'Erweckung in den ländlichen Regionen & Alpentälern',
    category: 'Erweckung & Land',
    author: 'Tobias aus Innsbruck',
    date: 'Vor 4 Tagen',
    description: 'Dass kein Dorf und kein Tal in Österreich vergessen bleibt – sondern das Licht des Evangeliums in jede Familie und jedes Haus scheint.',
    prayerCount: 37,
    supporters: []
  }
];

const PRAYER_GUIDES = [
  {
    title: '1. Anbetung & Dank (10 Min.)',
    verse: '„Kommt herzu, lasst uns dem HERRN frohlocken und jauchzen dem Hort unseres Heils!“ – Psalm 95,1',
    points: [
      'Danke Gott für Seine Treue und Gnade über Österreich.',
      'Preise Jesus als den Herrn über jedes Bundesland, jede Stadt und jedes Dorf.',
      'Stille dein Herz und richte deinen Fokus ganz auf Gottes Gegenwart.'
    ]
  },
  {
    title: '2. Buße & Reinigung (10 Min.)',
    verse: '„...und mein Volk, über das mein Name genannt ist, sich demütigt, dass sie beten und mein Angesicht suchen und sich von ihren bösen Wegen bekehren, so will ich vom Himmel her hören und ihre Sünde vergeben und ihr Land heilen.“ – 2. Chronik 7,14',
    points: [
      'Bitte um Vergebung für Gleichgültigkeit, Stolz und Lauheit in unseren eigenen Herzen.',
      'Stehe stellvertretend im Riss für das Land Österreich und bitte um Barmherzigkeit.'
    ]
  },
  {
    title: '3. Fürbitte für Erweckung in Österreich (25 Min.)',
    verse: '„Die Ernte ist groß, aber wenige sind der Arbeiter. Darum bittet den Herrn der Ernte, dass er Arbeiter in seine Ernte aussende.“ – Matthäus 9,37-38',
    points: [
      'Bete für eine tiefe Sehnsucht nach Gott unter den Menschen in Österreich.',
      'Bete für Mut und Vollmacht für Christen, das Evangelium im Alltag zu teilen.',
      'Bete für Familien, Ehen, Kinder und die junge Generation.'
    ]
  },
  {
    title: '4. Proklamation & Segen (15 Min.)',
    verse: '„Der HERR segne dich und behüte di

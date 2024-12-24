# ASU Course Search Application

## Setup Instructions

1. Create a new Vite project with React and TypeScript:
```bash
npm create vite@latest asu-course-search -- --template react-ts
cd asu-course-search
```

2. Install dependencies:
```bash
npm install @radix-ui/react-dialog @radix-ui/react-navigation-menu @radix-ui/react-badge cmdk framer-motion lucide-react three tailwindcss postcss autoprefixer tailwind-merge clsx class-variance-authority
```

3. Initialize Tailwind CSS:
```bash
npx tailwindcss init -p
```

## Project Structure

### 1. Types (src/types/course.ts)
```typescript
export interface Course {
  id: string;
  code: string;
  title: string;
  description: string;
  credits: number;
  prerequisites?: string[];
  department: string;
  level: "Undergraduate" | "Graduate";
  topics: string[];
}
```

### 2. Data (src/data/courses.ts)
```typescript
import { Course } from "@/types/course";

export const courses: Course[] = [
  {
    id: "1",
    code: "CSE 471",
    title: "Introduction to Artificial Intelligence",
    description: "Introduces fundamental concepts in artificial intelligence, including problem solving, knowledge representation, reasoning, planning, and machine learning.",
    credits: 3,
    prerequisites: ["CSE 310"],
    department: "Computer Science",
    level: "Undergraduate",
    topics: ["AI", "Machine Learning", "Neural Networks"],
  },
  {
    id: "2",
    code: "CSE 475",
    title: "Machine Learning",
    description: "Fundamental concepts in machine learning, including supervised and unsupervised learning, neural networks, and deep learning architectures.",
    credits: 3,
    prerequisites: ["CSE 310", "MAT 267"],
    department: "Computer Science",
    level: "Undergraduate",
    topics: ["AI", "Machine Learning", "Deep Learning"],
  },
  {
    id: "3",
    code: "CSE 571",
    title: "Advanced Artificial Intelligence",
    description: "Advanced topics in artificial intelligence, including natural language processing, computer vision, and robotics.",
    credits: 3,
    prerequisites: ["CSE 471"],
    department: "Computer Science",
    level: "Graduate",
    topics: ["AI", "NLP", "Computer Vision"],
  },
];

export const searchCourses = (query: string): Course[] => {
  const lowercaseQuery = query.toLowerCase();
  return courses.filter(course => 
    course.title.toLowerCase().includes(lowercaseQuery) ||
    course.description.toLowerCase().includes(lowercaseQuery) ||
    course.topics.some(topic => topic.toLowerCase().includes(lowercaseQuery)) ||
    course.code.toLowerCase().includes(lowercaseQuery)
  );
};
```

### 3. Components

#### SearchHeader (src/components/SearchHeader.tsx)
```typescript
import React from "react";
import {
  NavigationMenu,
  NavigationMenuItem,
  NavigationMenuLink,
  NavigationMenuList,
} from "@/components/ui/navigation-menu";
import { Button } from "@/components/ui/button";
import { UserCircle } from "lucide-react";

interface SearchHeaderProps {
  logoUrl?: string;
  navigationItems?: Array<{
    label: string;
    href: string;
  }>;
  onProfileClick?: () => void;
}

const SearchHeader = ({
  logoUrl = "https://www.asu.edu/asuthemes/5.0/assets/arizona-state-university-logo.png",
  navigationItems = [
    { label: "Home", href: "/" },
    { label: "Courses", href: "/courses" },
    { label: "Programs", href: "/programs" },
    { label: "About", href: "/about" },
  ],
  onProfileClick = () => {},
}: SearchHeaderProps) => {
  return (
    <header className="h-20 w-full bg-white border-b border-gray-200 px-6 flex items-center justify-between fixed top-0 z-50">
      <div className="flex items-center gap-8">
        <a href="/" className="flex items-center">
          <img src={logoUrl} alt="ASU Logo" className="h-12 object-contain" />
        </a>

        <NavigationMenu>
          <NavigationMenuList>
            {navigationItems.map((item) => (
              <NavigationMenuItem key={item.label}>
                <NavigationMenuLink
                  href={item.href}
                  className="px-4 py-2 text-gray-700 hover:text-[#8C1D40] transition-colors"
                >
                  {item.label}
                </NavigationMenuLink>
              </NavigationMenuItem>
            ))}
          </NavigationMenuList>
        </NavigationMenu>
      </div>

      <div className="flex items-center gap-4">
        <Button
          variant="ghost"
          size="icon"
          className="text-gray-700 hover:text-[#8C1D40] transition-colors"
          onClick={onProfileClick}
        >
          <UserCircle className="h-6 w-6" />
        </Button>
      </div>
    </header>
  );
};

export default SearchHeader;
```

#### SearchHero (src/components/SearchHero.tsx)
```typescript
import { useState, useEffect } from "react";
import {
  Command,
  CommandList,
  CommandEmpty,
  CommandGroup,
  CommandItem,
  CommandInput,
} from "@/components/ui/command";
import { Search } from "lucide-react";
import { searchCourses } from "@/data/courses";
import type { Course } from "@/types/course";
import { CourseDialog } from "./CourseDialog";

interface SearchHeroProps {
  onSearch?: (query: string) => void;
}

const SearchHero = ({ onSearch = () => {} }: SearchHeroProps) => {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState<Course[]>([]);
  const [selectedCourse, setSelectedCourse] = useState<Course | null>(null);
  const [dialogOpen, setDialogOpen] = useState(false);

  useEffect(() => {
    if (query.length >= 2) {
      const searchResults = searchCourses(query);
      setResults(searchResults);
    } else {
      setResults([]);
    }
  }, [query]);

  const handleSelect = (course: Course) => {
    setSelectedCourse(course);
    setDialogOpen(true);
    setQuery("");
    onSearch(course.title);
  };

  return (
    <div className="w-full min-h-[400px] bg-white flex flex-col items-center justify-center px-4 py-8">
      <h1 className="text-4xl font-bold text-[#8C1D40] mb-8 text-center">
        Find Your Perfect Course
      </h1>
      <div className="w-full max-w-2xl relative">
        <Command className="rounded-lg border shadow-md">
          <div className="flex items-center border-b px-3">
            <Search className="h-4 w-4 shrink-0 opacity-50" />
            <CommandInput
              placeholder="Search courses (e.g. 'artificial intelligence')"
              value={query}
              onValueChange={setQuery}
            />
          </div>
          {query.length >= 2 && (
            <CommandList>
              <CommandEmpty>No results found.</CommandEmpty>
              <CommandGroup heading="Courses">
                {results.map((course) => (
                  <CommandItem
                    key={course.id}
                    onSelect={() => handleSelect(course)}
                    className="cursor-pointer"
                  >
                    <div className="flex flex-col">
                      <div className="font-medium">
                        {course.code} - {course.title}
                      </div>
                      <div className="text-sm text-gray-500 truncate">
                        {course.description}
                      </div>
                    </div>
                  </CommandItem>
                ))}
              </CommandGroup>
            </CommandList>
          )}
        </Command>
      </div>

      <CourseDialog
        course={selectedCourse}
        open={dialogOpen}
        onOpenChange={setDialogOpen}
      />
    </div>
  );
};

export default SearchHero;
```

#### CourseDialog (src/components/CourseDialog.tsx)
```typescript
import { Course } from "@/types/course";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
import { Badge } from "@/components/ui/badge";

interface CourseDialogProps {
  course: Course | null;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export function CourseDialog({ course, open, onOpenChange }: CourseDialogProps) {
  if (!course) return null;

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="max-w-2xl">
        <DialogHeader>
          <DialogTitle className="text-2xl font-bold flex items-center gap-3">
            <span className="text-[#8C1D40]">{course.code}</span>
            <span className="text-gray-700">{course.title}</span>
          </DialogTitle>
        </DialogHeader>
        <div className="space-y-4">
          <div className="flex items-center gap-2">
            <Badge variant="secondary">{course.department}</Badge>
            <Badge variant="secondary">{course.level}</Badge>
            <Badge variant="secondary">{course.credits} Credits</Badge>
          </div>

          <p className="text-gray-700">{course.description}</p>

          {course.prerequisites && course.prerequisites.length > 0 && (
            <div>
              <h3 className="font-semibold mb-2">Prerequisites</h3>
              <div className="flex gap-2">
                {course.prerequisites.map((prereq) => (
                  <Badge key={prereq} variant="outline">{prereq}</Badge>
                ))}
              </div>
            </div>
          )}

          <div>
            <h3 className="font-semibold mb-2">Topics</h3>
            <div className="flex flex-wrap gap-2">
              {course.topics.map((topic) => (
                <Badge key={topic} variant="default" className="bg-[#8C1D40]">
                  {topic}
                </Badge>
              ))}
            </div>
          </div>
        </div>
      </DialogContent>
    </Dialog>
  );
}
```

#### FloatingCards (src/components/FloatingCards.tsx)
```typescript
import { useEffect, useRef, useState } from "react";
import * as THREE from "three";
import TrendingCard from "./TrendingCard";

interface FloatingCardsProps {
  cards?: Array<{
    topic: string;
    description: string;
  }>;
  onCardClick?: (topic: string) => void;
}

const FloatingCards = ({
  cards = [
    {
      topic: "Artificial Intelligence",
      description: "Explore classes about AI and machine learning",
    },
    {
      topic: "Data Science",
      description: "Learn statistical analysis and data visualization",
    },
    {
      topic: "Web Development",
      description: "Master modern web technologies and frameworks",
    },
    {
      topic: "Cybersecurity",
      description: "Study network security and ethical hacking",
    },
  ],
  onCardClick = () => {},
}: FloatingCardsProps) => {
  const containerRef = useRef<HTMLDivElement>(null);
  const [cardPositions, setCardPositions] = useState<Array<{ x: number; y: number }>>
    (cards.map(() => ({ x: 0, y: 0 })));

  useEffect(() => {
    if (!containerRef.current) return;

    let frameId: number;
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(
      75,
      containerRef.current.clientWidth / containerRef.current.clientHeight,
      0.1,
      1000
    );
    const renderer = new THREE.WebGLRenderer({ alpha: true });

    renderer.setSize(
      containerRef.current.clientWidth,
      containerRef.current.clientHeight
    );
    containerRef.current.appendChild(renderer.domElement);

    camera.position.z = 5;

    const orbitRadius = 3;
    const orbitSpeed = 0.001;
    let time = 0;

    const animate = () => {
      time += orbitSpeed;

      const newPositions = cards.map((_, index) => {
        const angle = (2 * Math.PI * index) / cards.length + time;
        const x = Math.cos(angle) * orbitRadius;
        const y = Math.sin(angle) * orbitRadius;
        return { x, y };
      });

      setCardPositions(newPositions);
      renderer.render(scene, camera);

      frameId = requestAnimationFrame(animate);
    };

    animate();

    return () => {
      if (containerRef.current) {
        containerRef.current.removeChild(renderer.domElement);
      }
      cancelAnimationFrame(frameId);
    };
  }, [cards.length]);

  return (
    <div
      className="relative w-[1200px] h-[500px] bg-gray-50"
      ref={containerRef}
    >
      {cards.map((card, index) => (
        <div
          key={index}
          className="absolute transition-all duration-500 ease-in-out"
          style={{
            transform: `translate(${cardPositions[index].x * 100 + 600}px, ${
              cardPositions[index].y * 50 + 250
            }px)`,
          }}
        >
          <TrendingCard
            topic={card.topic}
            description={card.description}
            onClick={() => onCardClick(card.topic)}
          />
        </div>
      ))}
    </div>
  );
};

export default FloatingCards;
```

#### TrendingCard (src/components/TrendingCard.tsx)
```typescript
import React from "react";
import { Card, CardContent } from "@/components/ui/card";
import { motion } from "framer-motion";

interface TrendingCardProps {
  topic?: string;
  description?: string;
  onClick?: () => void;
}

const TrendingCard = ({
  topic = "Artificial Intelligence",
  description = "Explore classes about AI and machine learning",
  onClick = () => {},
}: TrendingCardProps) => {
  return (
    <motion.div
      whileHover={{ scale: 1.05, y: -5 }}
      transition={{ type: "spring", stiffness: 300 }}
      className="cursor-pointer"
      onClick={onClick}
    >
      <Card className="w-[200px] h-[120px] bg-white border-2 hover:border-[#FFC627] transition-colors">
        <CardContent className="p-4 flex flex-col justify-between h-full">
          <h3 className="font-semibold text-[#8C1D40] text-lg truncate">
            {topic}
          </h3>
          <p className="text-sm text-gray-600 line-clamp-2">{description}</p>
        </CardContent>
      </Card>
    </motion.div>
  );
};

export default TrendingCard;
```

#### Home (src/components/home.tsx)
```typescript
import SearchHeader from "./SearchHeader";
import SearchHero from "./SearchHero";
import FloatingCards from "./FloatingCards";
import { searchCourses } from "@/data/courses";

function Home() {
  const handleSearch = (query: string) => {
    const results = searchCourses(query);
    console.log("Search results:", results);
  };

  const handleCardClick = (topic: string) => {
    const results = searchCourses(topic);
    console.log("Topic results:", results);
  };

  return (
    <div className="min-h-screen bg-gray-50">
      <SearchHeader />
      <main className="pt-20">
        <SearchHero onSearch={handleSearch} />
        <div className="flex justify-center mt-8">
          <FloatingCards onCardClick={handleCardClick} />
        </div>
      </main>
    </div>
  );
}

export default Home;
```

## Additional Setup

1. Update your `tailwind.config.js` to include the paths:
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

2. Update your `tsconfig.json` to include path aliases:
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

3. Add the following to your `src/index.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

This application features:
- A Google-inspired search interface for ASU courses
- Real-time search with autocomplete
- Floating cards with trending topics using Three.js
- Detailed course information in a modal dialog
- ASU branding colors (#8C1D40 maroon, #FFC627 gold)
- Responsive design with Tailwind CSS
- TypeScript for type safety
- Modern React patterns and hooks

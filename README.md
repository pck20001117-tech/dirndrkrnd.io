import {
  ArrowLeft,
  TrendingUp,
  TrendingDown,
  Users,
} from "lucide-react";

interface ResultsPageProps {
  data: {
    gender: string;
    exercise: string;
    grade: string;
    department: string;
    height: string;
    weight: string;
    sleepHours: string;
    bedtime: string;
  };
  onBack: () => void;
}

export function ResultsPage({
  data,
  onBack,
}: ResultsPageProps) {
  // 수면의 질 개선 점수 계산 로직
  const calculateSleepQualityScore = () => {
    let score = 50; // 기본 점수
    const factors: {
      factor: string;
      impact: number;
      reason: string;
    }[] = [];

    // 1. 운동 빈도 분석
    if (data.exercise === "안 함") {
      score += 20;
      factors.push({
        factor: "운동 빈도",
        impact: 20,
        reason:
          "평소 운동을 하지 않아 저녁 운동으로 적절한 피로감이 생겨 수면 유도에 도움이 됩니다",
      });
    } else {
      score += 5;
      factors.push({
        factor: "운동 빈도",
        impact: 5,
        reason:
          "이미 운동 습관이 있어 추가 효과는 제한적이지만 규칙적인 운동 시간 확보에 도움됩니다",
      });
    }

    // 2. 수면 시간 분석
    const sleepHours = parseFloat(data.sleepHours) || 7;
    if (sleepHours < 6) {
      score += 15;
      factors.push({
        factor: "수면 시간",
        impact: 15,
        reason:
          "수면 시간이 부족하여 저녁 운동으로 더 빠르고 깊은 수면에 들 수 있습니다",
      });
    } else if (sleepHours < 7) {
      score += 10;
      factors.push({
        factor: "수면 시간",
        impact: 10,
        reason:
          "수면 시간이 다소 부족하여 저녁 운동이 수면의 질 향상에 도움됩니다",
      });
    } else if (sleepHours >= 8) {
      score += 3;
      factors.push({
        factor: "수면 시간",
        impact: 3,
        reason:
          "충분한 수면을 취하고 있어 현재 상태 유지에 도움됩니다",
      });
    } else {
      score += 7;
      factors.push({
        factor: "수면 시간",
        impact: 7,
        reason:
          "적정 수면 시간을 유지하고 있어 저녁 운동이 수면 리듬 강화에 도움됩니다",
      });
    }

    // 3. 취침 시간 분석
    if (data.bedtime) {
      const bedtimeHour = parseInt(data.bedtime.split(":")[0]);
      if (bedtimeHour >= 0 && bedtimeHour < 6) {
        // 새벽 시간대
        score -= 10;
        factors.push({
          factor: "취침 시간",
          impact: -10,
          reason:
            "매우 늦은 취침 시간으로 저녁 운동 시 충분한 회복 시간이 부족할 수 있습니다",
        });
      } else if (bedtimeHour >= 23 || bedtimeHour < 6) {
        score += 12;
        factors.push({
          factor: "취침 시간",
          impact: 12,
          reason:
            "늦은 취침 습관이 있어 저녁 운동으로 더 일찍 잠들 수 있습니다",
        });
      } else if (bedtimeHour >= 22 && bedtimeHour < 23) {
        score += 8;
        factors.push({
          factor: "취침 시간",
          impact: 8,
          reason:
            "적절한 취침 시간을 유지하고 있어 저녁 운동이 수면 리듬 안정화에 도움됩니다",
        });
      } else {
        score += 5;
        factors.push({
          factor: "취침 시간",
          impact: 5,
          reason: "규칙적인 취침 패턴 형성에 도움이 됩니다",
        });
      }
    }

    // 4. BMI 분석
    if (data.height && data.weight) {
      const height = parseFloat(data.height) / 100;
      const weight = parseFloat(data.weight);
      const bmi = weight / (height * height);

      if (bmi < 18.5) {
        score += 5;
        factors.push({
          factor: "체질량 지수",
          impact: 5,
          reason:
            "저체중으로 저녁 운동이 식욕 증진과 숙면에 도움될 수 있습니다",
        });
      } else if (bmi >= 25) {
        score += 15;
        factors.push({
          factor: "체질량 지수",
          impact: 15,
          reason:
            "체중 관리가 필요하여 저녁 운동이 대사 활성화와 수면의 질 개선에 큰 도움이 됩니다",
        });
      } else {
        score += 7;
        factors.push({
          factor: "체질량 지수",
          impact: 7,
          reason:
            "정상 체중을 유지하고 있어 저녁 운동이 건강한 수면 패턴 유지에 도움됩니다",
        });
      }
    }

    // 5. 학년별 스트레스 분석
    if (data.grade === "4학년") {
      score += 10;
      factors.push({
        factor: "학년 (스트레스)",
        impact: 10,
        reason:
          "졸업을 앞두고 있어 스트레스가 높을 수 있으며, 저녁 운동이 스트레스 해소와 숙면에 도움됩니다",
      });
    } else if (data.grade === "1학년") {
      score += 8;
      factors.push({
        factor: "학년 (적응)",
        impact: 8,
        reason:
          "새로운 환경 적응 시기로 저녁 운동이 생활 리듬 형성에 도움됩니다",
      });
    } else {
      score += 5;
      factors.push({
        factor: "학년",
        impact: 5,
        reason:
          "저녁 운동이 규칙적인 생활 패턴 유지에 도움됩니다",
      });
    }

    // 점수를 1-100 범위로 제한
    score = Math.min(100, Math.max(1, score));

    return { score: Math.round(score), factors };
  };

  const { score, factors } = calculateSleepQualityScore();

  // 점수에 따른 색상
  const getScoreColor = (score: number) => {
    if (score >= 80) return "text-green-600";
    if (score >= 60) return "text-blue-600";
    if (score >= 40) return "text-yellow-600";
    return "text-orange-600";
  };

  const getScoreGradient = (score: number) => {
    if (score >= 80) return "from-green-500 to-emerald-500";
    if (score >= 60) return "from-blue-500 to-indigo-500";
    if (score >= 40) return "from-yellow-500 to-orange-500";
    return "from-orange-500 to-red-500";
  };

  return (
    <div className="space-y-6">
      {/* 헤더 */}
      <div className="bg-white rounded-2xl shadow-xl p-6">
        <button
          onClick={onBack}
          className="flex items-center gap-2 text-gray-600 hover:text-indigo-600 transition-colors mb-4"
        >
          <ArrowLeft className="w-5 h-5" />
          돌아가기
        </button>
        <h1 className="text-center text-indigo-600 mb-2">
          수면의 질 개선 분석 결과
        </h1>
        <p className="text-center text-gray-600">
          일주일간 저녁 운동 시 예상 효과
        </p>
      </div>

      {/* 총 인원 통계 */}
      <div className="bg-white rounded-2xl shadow-xl p-6">
        <div className="flex items-center gap-2 mb-4">
          <Users className="w-5 h-5 text-indigo-500" />
          <h2 className="text-gray-800">대전대학교 학생 수</h2>
        </div>
        <div className="space-y-3">
          <div className="flex justify-between items-center">
            <span className="text-gray-700">총 인원</span>
            <div className="flex items-center gap-3">
              <span className="text-gray-900">7,315명</span>
              <span className="text-gray-500">100%</span>
            </div>
          </div>
          <div className="w-full h-2 bg-gray-200 rounded-full overflow-hidden">
            <div className="h-full bg-indigo-500 w-full"></div>
          </div>

          <div className="flex justify-between items-center mt-4">
            <span className="text-gray-700">남학생</span>
            <div className="flex items-center gap-3">
              <span className="text-blue-600">4,228명</span>
              <span className="text-gray-500">57.8%</span>
            </div>
          </div>
          <div className="w-full h-2 bg-gray-200 rounded-full overflow-hidden">
            <div
              className="h-full bg-blue-500"
              style={{ width: "57.8%" }}
            ></div>
          </div>

          <div className="flex justify-between items-center mt-4">
            <span className="text-gray-700">여학생</span>
            <div className="flex items-center gap-3">
              <span className="text-pink-600">3,087명</span>
              <span className="text-gray-500">42.2%</span>
            </div>
          </div>
          <div className="w-full h-2 bg-gray-200 rounded-full overflow-hidden">
            <div
              className="h-full bg-pink-500"
              style={{ width: "42.2%" }}
            ></div>
          </div>
        </div>
      </div>

      {/* 수면의 질 점수 */}
      <div className="bg-white rounded-2xl shadow-xl p-8">
        <h2 className="text-center text-gray-800 mb-6">
          저녁 운동 시 수면의 질 개선도
        </h2>
        <div className="flex flex-col items-center">
          <div
            className={`relative w-48 h-48 rounded-full bg-gradient-to-br ${getScoreGradient(
              score,
            )} flex items-center justify-center shadow-2xl mb-6`}
          >
            <div className="absolute inset-2 bg-white rounded-full flex items-center justify-center">
              <div className="text-center">
                <div className={`${getScoreColor(score)}`}>
                  {score}점
                </div>
                <div className="text-gray-500 mt-1">
                  / 100점
                </div>
              </div>
            </div>
          </div>
          <div className="w-full bg-gray-200 rounded-full h-3 overflow-hidden">
            <div
              className={`h-full bg-gradient-to-r ${getScoreGradient(score)} transition-all duration-1000`}
              style={{ width: `${score}%` }}
            ></div>
          </div>
        </div>
      </div>

      {/* 상세 분석 */}
      <div className="bg-white rounded-2xl shadow-xl p-6">
        <h2 className="text-gray-800 mb-4">상세 분석</h2>
        <div className="space-y-4">
          {factors.map((factor, index) => (
            <div
              key={index}
              className="border-l-4 border-indigo-500 pl-4 py-2 bg-indigo-50 rounded-r-lg"
            >
              <div className="flex items-start justify-between mb-2">
                <span className="text-gray-800">
                  {factor.factor}
                </span>
                <div className="flex items-center gap-1">
                  {factor.impact > 0 ? (
                    <TrendingUp className="w-4 h-4 text-green-600" />
                  ) : (
                    <TrendingDown className="w-4 h-4 text-red-600" />
                  )}
                  <span
                    className={
                      factor.impact > 0
                        ? "text-green-600"
                        : "text-red-600"
                    }
                  >
                    {factor.impact > 0 ? "+" : ""}
                    {factor.impact}점
                  </span>
                </div>
              </div>
              <p className="text-gray-600">{factor.reason}</p>
            </div>
          ))}
        </div>
      </div>

      {/* 추천 사항 */}
      <div className="bg-gradient-to-br from-indigo-50 to-blue-50 rounded-2xl shadow-xl p-6 border-2 border-indigo-200">
        <h2 className="text-indigo-800 mb-3">💡 맞춤 추천</h2>
        <div className="space-y-2 text-gray-700">
          {score >= 70 ? (
            <>
              <p>
                ✓ 저녁 운동이 수면의 질 향상에 큰 도움이 될
                것으로 예상됩니다
              </p>
              <p>
                ✓ 저녁 식사 2시간 후, 취침 2-3시간 전 운동을
                권장합니다
              </p>
              <p>
                ✓ 가벼운 유산소 운동(산책, 조깅)으로
                시작해보세요
              </p>
            </>
          ) : score >= 50 ? (
            <>
              <p>
                ✓ 저녁 운동이 적당한 효과를 가져올 것으로
                예상됩니다
              </p>
              <p>
                ✓ 본인에게 맞는 운동 강도를 찾는 것이 중요합니다
              </p>
              <p>
                ✓ 규칙적인 시간에 운동하는 습관을 만들어보세요
              </p>
            </>
          ) : (
            <>
              <p>
                ✓ 저녁 운동이 제한적인 효과를 보일 수 있습니다
              </p>
              <p>
                ✓ 운동 시간대를 조정하거나 강도를 낮춰보세요
              </p>
              <p>✓ 수면 전문의와 상담을 고려해보세요</p>
            </>
          )}
        </div>
      </div>
    </div>
  );
}
